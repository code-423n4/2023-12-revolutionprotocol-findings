## CultureIndex.sol

### [G-01] Enum validation not needed

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L160

`&& uint8(metadata.mediaType) <= 5` can be removed from the `require` statement because the value of mediaType is already validated when the function parses the input `metadata` into a `ArtPieceMetadata` struct.

### [G-02] Increments in for loop won't overflow

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L185

Remove `i++` at L185 and add `unchecked { ++i }` after L187.

### [G-03] Duplicated `erc20VotingToken.totalSupply()` call

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L226-L230

`erc20VotingToken.totalSupply()` is called twice: L227 and L230. Call it only once and use a local variable to store the returned value.

### [G-04] Store creators as a hash

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L237

Storing the creators array can become very expensive with a few creators on the array. For example, 5 creators would cost a bit more than 100k gas. Reading each creator from storage also costs gas, although not as much.

It's possible to hash the creators array into a bytes32 hash that can be stored instead of the array itself. To ensure the data is easily available, the array could be emitted in an event, which is much cheaper than storing on chain. Afterwards when needed (for example during AuctionHouse's auction settlement), the array of creators would have to be provided as input and validated against the stored hash.

This optimization only makes sense if many creators per piece are expected. If the expected use is to have just a few creators per piece (let's say <= 2), then this optimization and the extra complexity that it carries might not be worth it.

### [G-05] Merge instances of the same for loop

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L185-L188
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L236-L238
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L243-L245

Inside `createPiece()`, the creators array is iterated 3 times (L185, L236 and L243). There is no need to do this. Move L236 and L243 for loops inside `validateCreatorsArray()`, join all of them and change the function name to reflect the new behavior.

### [G-06] Remove redundancy from events

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L240-L245

The event `PieceCreatorAdded` emits the sponsor once per creator. This seems redundant with emitting the sponsor in the `PieceCreated` event, which should be enough. Consider removing it to save gas.

### [G-07] Remove `ReentrancyGuardUpgradeable`

The reentrancy guard is protecting the contract against nothing. All functions that use `nonReentrant` are safe without the protection. Remove the reentrancy guard to reduce complexity and gas.

### [G-08] Remove redundant voter address validation in `_vote()`

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L309

There is no scenario in which this statement is true. At the time _vote() is invoked, either from _voteForMany() or from vote(), it's clear that the voter address won't be null. The reason is that it will be equal either to `msg.sender` or to a signer address which is checked inside `_verifyVoteSignature()` at L438.

### [G-09] Remove redundant vote weights stored on chain

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L317

`totalVoteWeights[pieceId]` is always equal to `MaxHeap.valueMapping(pieceId)`. Remove the former and rely only on the latter.

### [G-10] optimize loop in `batchVoteForManyWithSig()`

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L403-L409

No need to split the code in two loops. Join both for loops into one and use unchecked{ ++i } for increments.

### [G-11] Remove redundant signer address validation

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L441

`recoveredAddress == address(0)` can be removed, because it's redundant with the second condition `recoveredAddress != from`. Note that `from` is not zero at this point because of L438.

### [G-12] Remove redundant maxHeap size validation

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L487

This check is already performed inside `maxHeap.getMax()`. Remove it.

### [G-13] Avoid unnecessary `sload`s 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L522

Copying the entire `ArtPiece` struct from storage into memory doesn't seem to be the most efficient thing to do here, because each element of the struct is sloaded into memory, which costs gas. Instead, use `topVotedPieceId()` to simply retrieve `pieceId`, make the function return the id, not the piece, (if the dropper wants the piece info he can call getPieceById()) and adapt the code in `dropTopVotedPiece()` accordingly.

## ICultureIndex.sol

### [G-14] Don't store the keys of mapping inside the mapping's structs

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L116

`pieceId` in the `ArtPiece` struct is redundant. If you access a given piece in the pieces mapping then you already know the pieceId. Remove it from the struct data to save gas.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L132

The same happens in the `Vote` struct with `voterAddress`. Remove it from the struct data to save gas.

### [G-15] Pack and optimize variables in the `ArtPiece` struct

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L115-L125

- `creationBlock` can be stored in a smaller uint (for example uint40). This will pack the variable with `sponsor` and `isDropped`. (-20k gas per piece creation)
- the length of the `creators` array is bounded to the constant `MAX_NUM_CREATORS`, i.e. 100. This means that the length automatically stored as a uint256 can be optimized into a uint8. To do this, define creators as a mapping with keys mimicking and array and pack the uint8 length with the rest of the struct variables mentioned above. (-20k gas per piece creation)
- the data inside each `creators` element can also be packed. `bps` is bounded to 10_000 and therefore can comfortably fit into a uint96 to be packed with the creator's address in the `CreatorBps` struct. (-20k gas per creator per piece creation)

## MaxHeap.sol

### [G-16] Consider inheritance instead of composition for CultureIndex <> MaxHeap contracts

Consider merging the MaxHeap contract with the CultureIndex contract. This way, all external onlyAdmin functions can be replaced with internal functions to save gas and cut down on complexity.

### [G-17] Remove redundant position validation

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L79

`pos` is always checked to be nonzero before calling parent(). There is no need to require `pos != 0`. The require statement can be removed.

### [G-18] Consider merging `valueMapping` and `positionMapping`

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L69-L73C56

The position of an item in MaxHeap can be stored in a small uint. For example, using uint40 would handle up to 1 trillion simultaneous pieces. On the other hand, weights can probably be stored in a uint smaller than 256 bits. Therefore, using a single mapping and packing both variables together would save up to ~20k gas for the first voter of a piece and up to ~5k gas for subsequent ones.

### [G-19] Simplify `insert()`

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L119-L130

CultureIndex always calls insert() with value = 0. This means that:
- the input parameter `value` is not needed.
- the while loop will never execute and can be removed.

Also use unchecked for the size increment.

### [G-20] Remove redundant condition and unreachable code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L143-L150

Nothing to decide here. `newValue` is always greater than oldValue according to CultureIndex's logic, which means that the else block will never be executed. The if-else can be removed, leaving only the while loop.

### [G-21] Reorder logic in `maxHeapify()`

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L102

When the condition at L102 is met, the L95-L100 code would have been executed wastefully. To save gas, especially the `sload`s from L98-L100, place L102 at the beginning of the function.

### [G-22] Remove unused imports

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L5

The reentrancy guard is not used. Remove it to save on deployment costs.

## VerbsToken.sol

### [G-23] Consider inheritance instead of composition for VerbsToken <> CultureIndex contracts

The VerbsToken and CultureIndex contracts seem to belong together. VerbsToken simply adds ERC721 features to dropped art pieces. By splitting the logic in two contracts an overhead in gas is introduced and the codebase becomes unnecessarily complex. I recommend merging these contracts into one.

### [G-24] Remove redundant calls fetching the top voted piece

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L282-L292

The same `artPiece` is put into memory twice: L282 and L292. Remove L282 and simply use L292 `_artPiece`.

### [G-25] Remove redundant creators length validation

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L286-L289

This is redundant with CultureIndex.sol logic. It's not possible to create art pieces that violate this requirement. Remove it.

### [G-26] Don't duplicate data on chain

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L296-L308

It's expensive to store the art piece information in VerbsToken's contract. The art piece is already stored in CultureIndex and can't be modified there. Therefore, avoid duplicating this data and store a mapping of `verbId → pieceId` instead. If CultureIndex address is expected to change (the smart contract allows it through setCultureIndex), store the current CultureIndex address together with the pieceId (note that pieceId and address can probably be packed together).

### [G-27] Remove try-catch block

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L291-L318

Overriding cultureIndex.dropTopVotedPiece errors with a custom "dropTopVotedPiece failed" error seems like an overkill. Unless the minter is a smart contract that is expected to automatically react to this specific error (which I understand is not the case), I recommend removing the try-catch block to simplify the function.

### [G-28] Remove `ReentrancyGuardUpgradeable`

The reentrancy guard is protecting the contract against nothing. All functions that use `nonReentrant` seem safe without the protection. Remove the reentrancy guard to reduce complexity and gas.

## AuctionHouse.sol

### [G-29] Simplify `ethPaidTocreators` calculation

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L390-L391

The sum of creators bps is always 10_000, because it's required in CultureIndex. `ethPaidToCreators` can be calculated outside the for loop like so: ethPaidToCreators = creatorsShare * entropyRateBps / 10_000. This will also reduce rounding errors.

### [G-30] Optimize art piece getters

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L370-L371
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L385

In the `_settleAuction()` function there are multiple calls to `verbs.getArtPieceById(_auction.verbId)`. This is inefficient because it would be enough to call it once and because there are variables being read from storage (pieceId, metadata, isDropped, creationBlock, quorumVotes, totalERC20Supply, totalVotesSupply) that are not used. Fix this by calling the getter once and consider adding getters in VerbsToken so that AuctionHouse can access what it needs more efficiently.

## TokenEmitterRewards.sol

### [G-31] Remove redundant checks

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L18-L21

`computeTotalReward(msgValue)` is already called inside `_depositPurchaseRewards()`. In addition, the condition tested should always be false given the BPS constants defined in RewardSplits.sol. To save gas remove L18. If it gets removed, also consider deleting the TokenEmitterRewards contract and simply use RewardSplits instead (a whole abstract contract just for performing a subtraction seems like a big overkill).
