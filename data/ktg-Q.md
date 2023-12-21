### L-01: Function **CultureIndex.createPiece** has no way of validating duplicates art pieces

Function **createPiece** is currently implemented as follows:

```solidity
function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

        // Validate the media type and associated data
        validateMediaType(metadata);

        uint256 pieceId = _currentPieceId++;
....
```
As you can see, only the creators array and media type is checked, no duplication check. This could lead to an art piece is sold for more than once on Revolution Protocol


