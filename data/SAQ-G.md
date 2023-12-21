## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls | 6 | - |
| [G-02] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 1 | - |
| [G-03] | Public function not called by the contract should be declared external instead | 6 | - |
| [G-04] | Should use arguments instead of state variable  | 1 | - |
| [G-05] | Using storage instead of memory for structs/arrays saves gas | 8 | - |
| [G-06] | Duplicated require()/if() checks should be refactored to a modifier or function  | 2 | - |
| [G-07] | A modifier used only once and not being inherited should be inlined to save gas | 1 | - |
| [G-08] | Use selfbalance() instead of address(this).balance | 2 | - |
| [G-09] | abi.encode() is less efficient than  abi.encodepacked() | 1 | - |
| [G-10] | Using ERC721A instead of ERC721 for more gas-efficient  | 1 | - |
| [G-11] | Sort Solidity operations using short-circuit mode | 2 | - |
| [G-12] | Use hardcode address instead address(this) | 1 | - |
| [G-13] | Use assembly for math (add, sub, mul, div) | 2 | - |
| [G-14] | Expression `` is cheaper than new bytes(0) | 2 | - |
| [G-15] | Use uint256(1)/uint256(2) instead for true and false boolean states | 2 | - |
| [G-16] | Shorten the array rather than copying to a new one | 2 | - |
| [G-17] | Avoid fetching a low-level call's return data by using assembly | 1 | - |
| [G-18] | No need to evaluate all expressions to know if one of them is true | 3 | - |
| [G-19] | Assign the msg.value to varaible | 7 | - |
| [G-20] | Cache external calls outside of loop to avoid re-calling function on each iteration | 1 | - |

## Gas Optimizations  

## [G-01]  Avoid contract existence checks by using low level calls		

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence


```solidity
file: /packages/revolution/src/CultureIndex.sol

227            erc20VotingToken.totalSupply(),

228            erc721VotingToken.totalSupply()

322        maxHeap.updateValue(pieceId, totalWeight);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L227C2-L228C44


```solidity
file: /packages/revolution/src/AuctionHouse.sol

370                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;

371                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

454        if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L370C1-L371C83


## [G-02]  Expressions for constant values such as a call to keccak256(), should use immutable rather than constant


While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor. 


```solidity
file: /packages/revolution/src/CultureIndex.sol

29    bytes32 public constant VOTE_TYPEHASH =
        keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L29C1-L30C91


## [G-03]  Public function not called by the contract should be declared external instead 

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```solidity
file: /packages/revolution/src/MaxHeap.sol

119    function insert(uint256 itemId, uint256 value) public onlyAdmin {

136    function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {

169    function getMax() public view returns (uint256, uint256) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L119C1-L119C70


```solidity
file: /packages/revolution/src/CultureIndex.sol

209    function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L209C3-L212C33


```solidity
file: /packages/revolution/src/CultureIndex.sol

342    function voteForMany(uint256[] calldata pieceIds) public nonReentrant {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L342C1-L342C76


```solidity
file: /packages/revolution/src/ERC20TokenEmitter.sol

152    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L152C1-L156C82



## [G-04]  Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity
file: /packages/revolution/src/CultureIndex.sol

141        emit QuorumVotesBPSSet(quorumVotesBPS, _cultureIndexParams.quorumVotesBPS);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L141C1-L141C84


## [G-05] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function


```solidity
file: /packages/revolution/src/CultureIndex.sol
 
390   address[] memory from,
           uint256[][] calldata pieceIds,
           uint256[] memory deadline,
           uint8[] memory v,
           bytes32[] memory r,
395        bytes32[] memory s

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L390C1-L395C27


```solidity
file: /packages/revolution/src/AuctionHouse.sol

374                uint256[] memory vrgdaSplits = new uint256[](numCreators);

375                address[] memory vrgdaReceivers = new address[](numCreators);


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L374C1-L375C78


```solidity
file: /abstract/RewardSplits.sol

72        (RewardsSettings memory settings, uint256 totalReward) = computePurchaseRewards(paymentAmountWei);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L72C1-L72C107


## [G-06] Duplicated require()/if() checks should be refactored to a modifier or function   

to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /packages/revolution/src/CultureIndex.sol

///@audit the bellow require is duplicate on lines : 452 and 462

308        require(pieceId < _currentPieceId, "Invalid piece ID");

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L308C1-L308C64


```solidity
file: /packages/revolution/src/VerbsToken.sol

///@audit the bellow require is duplicate on line : 210

139        require(_minter != address(0), "Minter cannot be zero address");

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L139C1-L139C73


## [G-07] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file: /packages/revolution/src/MaxHeap.sol

41    modifier onlyAdmin() {
        require(msg.sender == admin, "Sender is not the admin");
        _;
    }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L41C1-L44C6


## [G-08] Use selfbalance() instead of address(this).balance

Using selfbalance() instead of address(this).balance can save gas in Solidity because selfbalance() is a built-in function that returns the balance of the current contract directly as a uint256 value, without the need to create a new address object.
On the other hand, using address(this).balance to get the balance of the current contract creates a new address object, which consumes more gas and can make the code less efficient.
By using selfbalance(), you can reduce the amount of gas consumed by your Solidity code and make your contracts more efficient.

```solidity
file: /packages/revolution/src/AuctionHouse.sol

348        if (address(this).balance < reservePrice) {

421        if (address(this).balance < _amount) revert("Insufficient balance");   

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L348C1-L348C52


## [G-09] abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

```solidity
file: /packages/revolution/src/CultureIndex.sol

431        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L431

## [G-10] Using ERC721A instead of ERC721 for more gas-efficient 

ERC721A is a proposed extension to the ERC721 standard that aims to improve gas efficiency and reduce the cost of deploying and using non-fungible tokens (NFTs) on the Ethereum blockchain. 

Reference 1: https://nextrope.com/erc721-vs-erc721a-2/

Reference 2 :https://github.com/chiru-labs/ERC721A#about-the-project

```solidity
file: /packages/revolution/src/CultureIndex.sol

20   contract CultureIndex is

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L20C1-L20C25


## [G-11] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows
 f(x) || g(y) 
 f(x) && g(y)
```

```solidity
file: /packages/revolution/src/AuctionHouse.sol

268        if (auction.startTime == 0 || auction.settled) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L268


```solidity
file: /packages/revolution/src/MaxHeap.sol

102        if (pos >= (size / 2) && pos <= size) return;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L102


## [G-12] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```

```solidity
file: /packages/revolution/src/AuctionHouse.sol

361            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L361C1-L361C86


## [G-13] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: packages/revolution/src/MaxHeap.sol

95        uint256 left = 2 * pos + 1;

96        uint256 right = 2 * pos + 2;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L95C1-L96C37


## [G-14] Expression `` is cheaper than new bytes(0)

```solidity
file: /packages/revolution/src/ERC20TokenEmitter.sol

191        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));

196            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L191


## [G-15] Use uint256(1)/uint256(2) instead for true and false boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

```solidity
file: /packages/revolution/src/CultureIndex.sol

63    mapping(uint256 => ArtPiece) public pieces;

526        pieces[piece.pieceId].isDropped = true;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L526


```solidity
file: /packages/revolution/src/VerbsToken.sol

221        isMinterLocked = true;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L221


## [G-16] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
file: /packages/revolution/src/AuctionHouse.sol

374                uint256[] memory vrgdaSplits = new uint256[](numCreators);

375                address[] memory vrgdaReceivers = new address[](numCreators);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L374C1-L375C78


## [G-17] Avoid fetching a low-level call's return data by using assembly

Even if you don't assign the call's second return value, it still gets copied to memory. Use assembly instead to prevent this and save 159 gas:

```solidity
(bool success,) = payable(receiver).call{gas: gas, value: value}("");

can be optimized to:
bool success;
assembly {
    success := call(gas, receiver, value, 0, 0, 0, 0)
}

```

```solidity
file: /packages/revolution/src/ERC20TokenEmitter.sol
 
191        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L191


## [G-18] No need to evaluate all expressions to know if one of them is true

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved

```solidity
file: /src/abstract/RewardSplits.sol

30        if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");

41        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L30C1-L30C120


```solidity
file: /packages/revolution/src/MaxHeap.sol

104        if (posValue < leftValue || posValue < rightValue) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L104C1-L104C61


## [G-19] Assign the msg.value to varaible

Assigning msg.value to another variable allows you to avoid repeated accesses to the msg.value special variable, which can potentially save gas. By assigning it to a local variable, you can reuse that variable throughout your function without incurring the cost of accessing msg.value multiple times.

```solidity
file: /packages/revolution/src/ERC20TokenEmitter.sol

160        require(msg.value > 0, "Must send ether");

166            msg.value,

221            msg.value,

223            msg.value - msgValueRemaining,

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L160


```solidity
file: /packages/revolution/src/AuctionHouse.sol

179        require(msg.value >= reservePrice, "Must send at least reservePrice");

181            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),

187        auction.amount = msg.value;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L179C1-L179C79



## [G-20] Cache external calls outside of loop to avoid re-calling function on each iteration

```solidity
file: /packages/revolution/src/AuctionHouse.sol

385                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L385C1-L385C118


