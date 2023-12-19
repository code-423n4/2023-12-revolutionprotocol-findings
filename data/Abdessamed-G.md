- On `CultureIndex.createPiece`, the [following loop](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L243) is only emitting an event, and that event emission can be done in the [loop above it](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L236) which saves some gas
```solidity
for (uint i; i < creatorArrayLength; i++) {
    newPiece.creators.push(creatorArray[i]);
    // Emit an event for each creator
    emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
}

emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
```
- on `CultureIndex.batchVoteForManyWithSig`, the [following loop logic](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L407) can be included in the [loop above it](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L403) which saves some gas. The gas saved can be notable if the `from` array provided as an argument is large: 
```solidity
 for (uint256 i; i < len; i++) {
    if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();
    _voteForMany(pieceIds[i], from[i]);
}
```