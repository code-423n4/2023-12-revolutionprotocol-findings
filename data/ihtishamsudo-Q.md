## Executive Summary 

Following Quality Assurance Reports Are The Notes, Refactoring Recommendations, And Some Low Severity bugs Found During the Auditing process.


## Table of Contents
* 1. [L- Not using OZ's ECDSA to ensure signature is safe from malleablity issues](#L-NotUsingOZsECDSAToEnsureSignatureMalleability)
* 2. [L - ```CultureIndex#voteForMany``` does not emit events, so it is difficult to track changes upon succesfull execution](#L-voteForManyDoesNotEmitEVent)
* 3. [L - Use of Push Mechanism For Transferring Creators Share Can Result In DoS](#L-PushCanResultDos)
* 4. [NC - Comments Contradicts/Mismatch](#NC-CommentsContradicts)


## 1. <a name='L-NotUsingOZsECDSAToEnsureSignatureMalleability'></a> L- Not using OZ's ECDSA to ensure signature is safe from malleablity issues
[CultureIndex#L435](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L435)

Even though the given function [_verifyVoteSignature](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L435) is implementing OZ's EIP712Upgradeable's ```_hashTypedDataV4``` to returns the hash of the fully encoded EIP712 message for this domain. But Not using OZ's ECDSA to obtain the signer(recoveredAddress) Even it's recommended by OZ to use ECDSA alongwith _hashTypedDataV4 to prevent malleablitiy issues because using ```ecrecover``` is exposed to malleability vulnerability.
Here is OZ recommendation in EIP712Upgradeable Library 
```
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
```
#### Recommendation 
Use OZ's ECDSA to prevent malleablitiy issues and also to follow best practices.

```diff
function _verifyVoteSignature(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal returns (bool success) {
        require(deadline >= block.timestamp, "Signature expired");  //@audit-info blocktimestamp check must.

        bytes32 voteHash;
        // malicious actor can replay signature on other networks due to absence of chainid
        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));
        //@audit-issue vote can be replay if fail?
        bytes32 digest = _hashTypedDataV4(voteHash);
        // ECDSA
-        address recoveredAddress = ecrecover(digest, v, r, s);
+        address recoveredAddress = ECDSA.recover(digest, v, r, s);

        // Ensure to address is not 0   //@audit typo (from)
        if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

        return true;
    }
```

## 2. <a name='L-voteForManyDoesNotEmitEVent'></a> L - ```voteForMany``` does not emit events, so it is difficult to track changes upon succesfull execution
[CultureIndex.sol#L353](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L353)

```CultureIndex#voteForMany``` is used for casting a vote for a list of artPiece. But it does not emiting event for tracking changes off chain & for succesfull execution confirmation. 
[_vote](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L323) is emiting voteCast Event but 
[_voteForMany](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L353) doesn't.

[initialize](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L113) should also emit events.
#### Recommendation 
```diff
function _voteForMany(uint256[] calldata pieceIds, address from) internal {
        uint256 len = pieceIds.length;
        for (uint256 i; i < len; i++) {
            _vote(pieceIds[i], from);
        }
+       emit VoteCast(pieceId, voter, weight, totalWeight);   
    }
```
```diff
function initialize(
    address _erc721Token,
    address _erc20TokenEmitter,
    address _initialOwner,
    address _weth,
    IRevolutionBuilder.AuctionParams calldata _auctionParams
) external initializer {
    
    ...code 

    
    verbs = IVerbsToken(_erc721Token);
    erc20TokenEmitter = IERC20TokenEmitter(_erc20TokenEmitter);
    timeBuffer = _auctionParams.timeBuffer;
    reservePrice = _auctionParams.reservePrice;
    minBidIncrementPercentage = _auctionParams.minBidIncrementPercentage;
    duration = _auctionParams.duration;
    creatorRateBps = _auctionParams.creatorRateBps;
    entropyRateBps = _auctionParams.entropyRateBps;
    minCreatorRateBps = _auctionParams.minCreatorRateBps;
    WETH = _weth;

    // Emit events for the values being set
+    emit TimeBufferSet(timeBuffer);
+    emit ReservePriceSet(reservePrice);
+    emit MinBidIncrementPercentageSet(minBidIncrementPercentage);
+    emit DurationSet(duration);
+    emit CreatorRateBpsSet(creatorRateBps);
+    emit EntropyRateBpsSet(entropyRateBps);
+    emit MinCreatorRateBpsSet(minCreatorRateBps);
+    emit WETHSet(WETH);
}

```

## 3. <a name='L-PushCanResultDos'></a> L - Use of Push Mechanism For Transferring Creators Share Can Result In DoS
[AuctionHouse.sol#L383-L396](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L383-L396)
[CultureIndex.sol#L237](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L237)

The ```_settle``` function is responsible for transferring shares to each creator. But it is using [push mechanism](https://youtu.be/8fNNVQv4-oY?t=1020) for transferring shares to each creator. This function iterates through arrays of creators and percentages, calculating the share to transfer to each creator based on their percentage. While the function's purpose is to fairly distribute tokens, a potential vulnerability arises when dealing with a large number of creators and percentages.
```solidity
//Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                if (creatorsShare > 0 && entropyRateBps > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
                        vrgdaReceivers[i] = creator.creator;
                        vrgdaSplits[i] = creator.bps;

                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
                        ethPaidToCreators += paymentAmount;
                        //@audit-issue using push in for loop
                        //Transfer creator's share to the creator
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
                    }
                }
```
#### Recommendation 
Try not to use Push Mechanism and Implement a batching mechanism that processes a limited number of winners and percentages in each iteration of the loop.

P.s It reports totally different issue (use of push mechanism) then Bot L2 which just report the use of for loop in public or external function results in DoS. Consider not flagging it as just a bot finding.

## 4. <a name='NC-CommentsContradicts'></a> Comments Contradicts/Mismatch
[MaxHeap.sol#L64](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L64)

```solidity
/// @notice Struct to represent an item in the heap by it's ID  //@audit-issue NC Typo 
    mapping(uint256 => uint256) public heap;
```
[MaxHeap.sol#L38-L44](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L38-L44)
```solidity
/**
     * @notice Require that the minter has not been locked.
     */     //@audit-issue comment contradict
    modifier onlyAdmin() {
        require(msg.sender == admin, "Sender is not the admin");
        _;
    }
```
[ERC20TokenEmitter.sol#L209-L215](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L209-L215)
```solidity
for (uint256 i = 0; i < addresses.length; i++) {
            if (totalTokensForBuyers > 0) {
                // transfer tokens to address       //@audit-issue NC type addresses
                _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
            }
            bpsSum += basisPointSplits[i];
        }
```
[CultureIndex.sol#L437-L438](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L437-L438)
```solidity
        // Ensure to address is not 0   //@audit typo (from)
        if (from == address(0)) revert ADDRESS_ZERO();
```
[CultureIndex.sol#L260-L267](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L260-L267)
```solidity
/**
     * @notice Returns the voting power of a voter at the current block.
     * @param account The address of the voter.
     * @return The voting power of the voter.
     */ //@audit-issue NC Typo Not returning voting power at current block 
    function getVotes(address account) external view override returns (uint256) {
        return _getVotes(account);
    }
```