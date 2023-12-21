
## [G-01] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

Total instances: 1
```
 bytes32 public constant VOTE_TYPEHASH =
        keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L29C4-L30C91

## [G‑02]Consider using ERC721A instead of ERC721

Total instances: 1
```
import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L22C1-L22C76

## [GAS-03] internal functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

Total instances: 1
```
 function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L179C4-L179C105

## [G-04] Unchecking arithmetics operations that can’t underflow/overflow
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block: 
Reference:
https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic

As the following only concerns substractions with owner supplied value, it’s not a stretch to consider that they are safe/trusted enough be wrapped with an unchecked block (around 25 gas saved per instance):

Total instances: 1
```
   uint256 creatorsShare = _auction.amount - auctioneerPayment;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L368C14-L368C77

## [G-05] Multiple accesses of a mapping/array should use a storage pointer
Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling proposals[_proposalId], save its reference via a storage pointer: Proposal storage proposal_ = proposals[_proposalId] and use the pointer instead.

Total instances: 4
```
emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L244C13-L244C103

```
  return votes[pieceId][voter].voterAddress != address(0);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L257C7-L257C65

```
    require(!pieces[pieceId].isDropped, "Piece has already been dropped");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L310C5-L310C79

```
    require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L311C5-L311C87


## [G-06] Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

Total instances: 2
```
  function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

        // Validate the media type and associated data
        validateMediaType(metadata);

        uint256 pieceId = _currentPieceId++;

        /// @dev Insert the new piece into the max heap
        maxHeap.insert(pieceId, 0);

        ArtPiece storage newPiece = pieces[pieceId];
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L209C3-L223C53

```
 function _mintTo(address to) internal returns (uint256) {
        ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

        // Check-Effects-Interactions Pattern
        // Perform all checks
        require(
            artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
            "Creator array must not be > MAX_NUM_CREATORS"
        );

        // Use try/catch to handle potential failure
        try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
            artPiece = _artPiece;
            uint256 verbId = _currentVerbId++;

            ICultureIndex.ArtPiece storage newPiece = artPieces[verbId];
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L281C4-L296C73

## [G-07] Caching global variables is expensive than using the variable itself.

Total instances: 4
```
  newPiece.sponsor = msg.sender;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L232C7-L232C39

```
  newPiece.creationBlock = block.number;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L233C7-L233C47

```
  startTime = block.timestamp;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L105C7-L105C37

```
         uint256 startTime = block.timestamp;
            uint256 endTime = startTime + duration;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L314C4-L315C52


## [G-08] abi.encode() is less efficient than abi.encodePacked()

Total instances: 1
```
   voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L431C6-L431C99