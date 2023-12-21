## [ G-01 ] Duplicates for loops can be combined as one


In [CultureIndex.sol#L236-L245](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L236-L245),

```
  for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

        // Emit an event for each creator
        for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }
```

Can be written as below:
```diff
  for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
++           // Emit an event for each creator
++            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

--        // Emit an event for each creator
--        for (uint i; i < creatorArrayLength; i++) {
--            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
--        }
```




In [CultureIndex.sol#L403-L408](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L403-L408),
```
 for (uint256 i; i < len; i++) {
            if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();
        }

        for (uint256 i; i < len; i++) {
            _voteForMany(pieceIds[i], from[i]);
        }
```

This can be written as below:
```diff
 for (uint256 i; i < len; i++) {
            if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();
++            _voteForMany(pieceIds[i], from[i]);
        }

--        for (uint256 i; i < len; i++) {
--            _voteForMany(pieceIds[i], from[i]);
--        }
        
```



## [G-02] Move checks to top of functions to prevent wasted gas on calculations.



In [CultureIndex.sol#L419-L444](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L419-L444),

```diff
 function _verifyVoteSignature(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal returns (bool success) {
++        // Ensure to address is not 0
++        if (from == address(0)) revert ADDRESS_ZERO();
        require(deadline >= block.timestamp, "Signature expired");

        bytes32 voteHash;

        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

        bytes32 digest = _hashTypedDataV4(voteHash);

        address recoveredAddress = ecrecover(digest, v, r, s);

--        // Ensure to address is not 0
--        if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

        return true;
    }
```
Checks should move to the top of the function to revert early in case `from == address(0)`.



## [G-03] Duplicate checks should be removed to save some gas

In [CultureIndex.sol#L486-L491](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L486-L491),
```diff
function topVotedPieceId() public view returns (uint256) {
--        require(maxHeap.size() > 0, "Culture index is empty");
        //slither-disable-next-line unused-return
        (uint256 pieceId, ) = maxHeap.getMax();
        return pieceId;
    }

```
Noted that `require(maxHeap.size() > 0, "Culture index is empty");` should be removed as the `size` variable is already checked in `maxHeap.getMax()` function.
