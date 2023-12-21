## [L-01] Callers/buyers can control `protocolRewardsRecipients` to receive self-rebates
The `ERC20TokenEmitter.buyToken()` function is designed in such a way that the caller (or buyer) can fully control the `protocolRewardsRecipients` parameter, which includes the `builder`, `purchaseReferral`, and `deployer` addresses. This design allows the caller to potentially assign all these addresses to themselves.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L152-L170

```solidity
    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        //prevent treasury from paying itself
        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

        require(msg.value > 0, "Must send ether");
        // ensure the same number of addresses and bps
        require(addresses.length == basisPointSplits.length, "Parallel arrays required");

        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );
```
Given this setup, if a caller assigns all three addresses (builder, purchaseReferral, deployer) to themselves, they could indeed benefit from the rewards percentages assigned to each of these roles. In the code, the basis points (BPS) for each reward are defined as constants:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L17-L21

- BUILDER_REWARD_BPS = 100 BPS (0.1%)
- PURCHASE_REFERRAL_BPS = 50 BPS (0.05%)
- DEPLOYER_REWARD_BPS = 25 BPS (0.025%)

When summed up, these total to 1.75% (175 BPS). So, if the buyer sets themselves as the recipient for all these rewards, they would effectively be receiving a 1.75% reward on their purchase value.

This could be a feature or a vulnerability, depending on the intended use and design of the contract:

1. **Feature**: If this mechanism is intentional, it might be designed to incentivize certain behaviors, like encouraging users to participate more actively in the ecosystem or rewarding them for different roles they play.
2. **Vulnerability**: If this was not an intended use case, it could be exploited by users to unjustly reward themselves, undermining the fairness of the reward distribution system.

To address this, if it's deemed a vulnerability, the smart contract could be updated to include checks that prevent the same address from being used for all these roles or to implement a more robust system for assigning these rewards. This requires careful consideration of the contract's intended economics and security implications.

## [L-02] Irreversible correction if `minCreatorRateBps` has been set too high
`AuctionHouse.setMinCreatorRateBps()` require new min rate cannot be lower than previous min rate:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L237-L241

```solidity
        //ensure new min rate cannot be lower than previous min rate
        require(
            _minCreatorRateBps > minCreatorRateBps,
            "Min creator rate must be greater than previous minCreatorRateBps"
        );
```
If it's has been set too high, whether deliberately or accidentally, there's no way to drop the threshold. Hence, this could affect future setting of a new `_creatorRateBps`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L218-L221

```solidity
        require(
            _creatorRateBps >= minCreatorRateBps,
            "Creator rate must be greater than or equal to minCreatorRateBps"
        );
```
## [L-03] Addressing Zero-Value Bids in Auction Contracts
The auction contract, as currently designed, presents a a low to medium severity issue due to its allowance for bids of 0 ETH under specific conditions. When the `_createAuction()` function initializes an auction, it sets `auction.amount` to 0 ETH. Combined with a `reserve price` also set at 0 ETH, this configuration allows the first bid to be 0 ETH, which satisfies the contract's require statements for both the reserve price and the minimum bid increment. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L179-L183

```solidity
        require(msg.value >= reservePrice, "Must send at least reservePrice");
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );
```
Subsequent bids can also be 0 ETH, maintaining the same auction amount. This design flaw could lead to an auction proceeding and closing without any financial transaction, contradicting the fundamental purpose of an auction to facilitate competitive bidding and sell items for the highest possible price. Implementing safeguards such as a non-zero minimum reserve price, a required minimum first bid, or dynamic bid increments would be essential to ensure the auction's functionality and integrity.

## [L-04] Auction Extension Mechanism and Ethereum Transaction Dynamics
The auction extension mechanism in Ethereum-based smart contracts, particularly when combined with the `minBidIncrementPercentage`, presents a low to medium severity issue due to its interaction with the dynamics of Ethereum transactions, including frontrunning and mempool observation. Designed to prevent sniping, the extension mechanism ensures fairness by allowing bids within a `timeBuffer` to prolong the auction. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L191-L192

```solidity
        bool extended = _auction.endTime - block.timestamp < timeBuffer;
        if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
```
However, this can lead to unpredictability in auction closure, potentially deterring bidders with strict budgets or timelines. Moreover, the visibility of transactions in the mempool before confirmation enables frontrunning, where opportunistic bidders can outbid others by observing their transactions. This scenario can result in auctions closing at lower bids than potentially achievable, as illustrated in the example where an auction expected to close at 11 ETH ends at 10.5 ETH due to frontrunning and extension. 

For example, if the current `_auction.amount` is 10 ETH, and `minBidIncrementPercentage` is 5%. Alice in the last moment attempts to bid 11 ETH. Bob, seeing this in the mempool, frontruns with 10.5 ETH. Alice's call is denied, as 10.5 x 1.05 = 11.025 ETH. `bool extended` is turned on and the auction is extended for a few minutes (`timeBuffer`) but Alice is no longer interested since her budget is 11 ETH. An auction could have been sold for 11 ETH ends up with 10.5 ETH.

The strategic behavior of bidders may also evolve, leading to less transparent and more complex bidding processes. Mitigating these issues could involve implementing secret bids with a reveal phase, employing anti-frontrunning techniques, or adjusting the `minBidIncrementPercentage` dynamically. While these measures aim to balance fairness, predictability, and transactional efficiency, they also introduce their own complexities and trade-offs, making this a nuanced issue requiring careful consideration in smart contract design.

## [L-05] Impact of Ether Value on Auction Bidding Mechanism
In the `createBid` function of the `AuctionHouse` smart contract, the `minBidIncrementPercentage` parameter, which sets the minimum bid increment in whole percentage points, could significantly impact the auction dynamics, especially in the context of a high Ether value. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L180-L183

```solidity
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );
```
Apparently, the denominator of `100` signifies the lowest minBidIncrementPercentage amount it could go is 1%.

As the worth of Ether increases, a minimum 1% bid increment translates into a larger absolute monetary value, potentially discouraging smaller bidders due to the higher financial commitment required for each subsequent bid. This reduced bid granularity could lead to fewer overall bids and might result in bidders overcommitting, as they are forced to raise their bids by at least this minimum percentage. The situation could alter the strategic approach to bidding, with participants possibly engaging in last-minute bidding wars or being deterred from participating if the increments exceed their budget constraints. 

To ensure a balanced and accessible auction environment, considering a more flexible increment system, such as smaller percentage increments (via the adoption of BPS, e.g. 10_000 is equivalent to 100%) or a maximum increment cap in Ether, might be beneficial to accommodate varying Ether values and maintain efficient market pricing.

## [L-06] Considerations for Hardcoding Addresses in Smart Contracts
In the `_settleAuction` function of the `AuctionHouse` smart contract, setting the `builder` and `purchaseReferral` fields to `address(0)` within the `IERC20TokenEmitter.ProtocolRewardAddresses` struct raises important considerations. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L403-L407

```solidity
                        IERC20TokenEmitter.ProtocolRewardAddresses({
                            builder: address(0),
                            purchaseReferral: address(0),
                            deployer: deployer
                        })
```
This design choice might align with the contract's current requirements if no rewards are intended for builders or referrers. However, it potentially limits future flexibility and adaptability to new features. While it might offer gas savings, the implications on the contract's functionality and the ecosystem should be carefully evaluated. Moreover, such hardcoding should be transparently documented for clarity and understanding. The use of upgradeable contract patterns provides some leeway for future modifications, but it's crucial to balance immediate simplicity against long-term contract evolution and security.

## [L-07] Challenges and Strategies in Managing Voting Participation in Growing Communities
As communities involved in cultural indexing and art piece voting grow, they face the challenge of declining active voter participation, which can hinder critical processes such as achieving quorum. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L523

```solidity
        require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");
```
This trend, often due to factors like reduced interest or lack of awareness, necessitates strategic solutions. The protocol can boost engagement through improved communication, incentives for participation, and streamlined voting processes. Additionally, adapting quorum requirements, i.e. `dynamic quorum adjustment` to reflect actual participation rates and carefully balancing tokenomics to avoid concentrated voting power are crucial. These measures, coupled with a focus on fostering long-term community involvement, are key to ensuring that every member feels their contribution is both meaningful and impactful in the evolving landscape of decentralized governance.

## [L-08] Inaccurate use of inequality operator
In the `getArtPieceById` function within the `VerbsToken` smart contract, the condition require`(verbId <= _currentVerbId, "Invalid piece ID")` should ideally use `<` instead of `<=`. This is because `_currentVerbId` is post-incremented in the `_mintTo` function,

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L294

```solidity
            uint256 verbId = _currentVerbId++;
```
which means that at any given point, `_currentVerbId` represents the next ID to be assigned, not an ID that has already been assigned to an existing NFT.

When the `_mintTo` function is called, it increments `_currentVerbId` after assigning the current ID to a new NFT. Therefore, the highest valid `verbId` at any moment is `_currentVerbId - 1`. If `getArtPieceById` is called with `verbId` equal to `_currentVerbId`, it refers to an ID that has not yet been assigned to an NFT, leading to a potential reference to a non-existent art piece.

Consider making the following change to ensure that the function only processes requests for IDs that have already been assigned to minted NFTs, thus maintaining the integrity of the function and avoiding potential errors or unexpected behavior.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L273-L276

```diff
    function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
-        require(verbId <= _currentVerbId, "Invalid piece ID");
+        require(verbId < _currentVerbId, "Invalid piece ID");
        return artPieces[verbId];
    }
```
## [L-09] Potential Risks in Dynamic NFT Metadata Management in `VerbsToken` Smart Contract
The `VerbsToken` smart contract contains two critical functions, [setContractURIHash](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L165-L171) and [setDescriptor](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L226-L236), that pose potential risks due to their ability to alter contract-level and individual NFT metadata, respectively. 

The `setContractURIHash` function, with a low to medium severity level, allows changing the collection-level metadata, which could impact the overall perception and value of the NFTs in the market. This might lead to confusion or mistrust among NFT owners and potential buyers if the collection's description or theme is altered significantly. 

On the other hand, the `setDescriptor` function poses a high-severity risk, as it directly affects the `tokenURI` of each NFT. Changes made by this function can be substantial as the `tokenURI` typically points to a JSON file that contains the NFTs' appearance and features, potentially compromising their originality and authenticity. This could have severe implications for the NFT's value and the owner's rights.

The presence of a [lockDescriptor](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L238-L246) function, which irreversibly prevents further changes to the descriptor, shows an awareness of the potential risks associated with changing NFT metadata. However, the absence of a similar lock function for the `contractURIHash` indicates a different level of consideration for the collection-level metadata compared to the individual NFT metadata.

To mitigate these risks, it is recommended to implement immutable metadata practices, enhance transparency and community involvement in any changes, provide clear documentation, and introduce a versioning system for metadata. 

## [L-10] `ECDSA.recover` over `ecrecover`
One of the most critical aspects to note about `ecrecover` is its vulnerability to malleable signatures. This means that a valid signature can be transformed into a different valid signature without needing access to the private key. Where possible, adopt `ECDSA.recover` as commented by the imported `EIP712Upgradeable` in CultureIndex.sol.

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/cryptography/EIP712Upgradeable.sol#L93-L110

```solidity
    /**
     * @dev Given an already https://eips.ethereum.org/EIPS/eip-712#definition-of-hashstruct[hashed struct], this
     * function returns the hash of the fully encoded EIP712 message for this domain.
     *
     * This hash can be used together with {ECDSA-recover} to obtain the signer of a message. For example:
     *
     * ```solidity
     * bytes32 digest = _hashTypedDataV4(keccak256(abi.encode(
     *     keccak256("Mail(address to,string contents)"),
     *     mailTo,
     *     keccak256(bytes(mailContents))
     * )));
     * address signer = ECDSA.recover(digest, signature);
     * ```
     */
    function _hashTypedDataV4(bytes32 structHash) internal view virtual returns (bytes32) {
        return MessageHashUtils.toTypedDataHash(_domainSeparatorV4(), structHash);
    }
``` 
Here's a specific instance entailed:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L435

```solidity
        address recoveredAddress = ecrecover(digest, v, r, s);
```
## [NC-01] Unutilized function
`CultureIndex.hasVoted`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L250-L258

```solidity
    /**
     * @notice Checks if a specific voter has already voted for a given art piece.
     * @param pieceId The ID of the art piece.
     * @param voter The address of the voter.
     * @return A boolean indicating if the voter has voted for the art piece.
     */
    function hasVoted(uint256 pieceId, address voter) external view returns (bool) {
        return votes[pieceId][voter].voterAddress != address(0);
    }
``` 
could have been used in the following require statement:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L311

```diff
-        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
+        require(!(hasVoted(pieceId, voter)), "Already voted");
```
## [NC-02] Comment and doc spec mismatch
On https://www.desmos.com/calculator/im67z1tate, the integral of price is supposed to be `p(x) = p0 * (1 - k)^(t - x/r)`. However, the exponent varies on the formula adopted by the protocol:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L85

```solidity
    // given # of tokens sold, returns integral of price p(x) = p0 * (1 - k)^(x/r) 

## [NC-01] Typo mistakes
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L64

```diff
-    /// @notice Struct to represent an item in the heap by it's ID
+    /// @notice Struct to represent an item in the heap by its ID
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L64-L65

```diff
-    /// @notice Struct to represent an item in the heap by it's ID
    mapping(uint256 => uint256) public heap;
+    /// @notice Mapping to represent an item in the heap by it's ID
    mapping(uint256 => uint256) public heap;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L437-L438

```diff
-        // Ensure to address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();
+        // Ensure from address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();
```
## [NC-03] Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.22",
settings: {
 optimizer: {
   enabled: true,
   runs: 1000,
 },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.