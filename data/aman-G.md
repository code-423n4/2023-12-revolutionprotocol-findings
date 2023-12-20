## [G-01] Caching the length of calldata increases gas cost instead of reducing it
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L354

Caching length of `pieceIds` in `_voteForMany` increase gs cost by 4 units in case of single `pieceIds` in other cases the cost remain same
```diff
function _voteForMany(uint256[] calldata pieceIds, address from) internal {
-  uint256 len = pieceIds.length;
+        for (uint256 i; i < pieceIds.length; i++) {
            _vote(pieceIds[i], from);
        }
    }
```

## [G-02] No need to iterate over a loop for event `PieceCreatorAdded` in `createPiece` function
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L243-L245

There is no need to iterate over loop for event in  `createPiece` function it can be handle in first loop at line https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L236
the suggested change will save 88 uint per iteration.
```diff
  function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        ...

           for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
+           emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }
        ...
-          for (uint i; i < creatorArrayLength; i++) {
-           emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
-       }
```