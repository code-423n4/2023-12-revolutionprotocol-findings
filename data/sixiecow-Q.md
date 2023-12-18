### reverting many transaction for a single one that failed
in the `CultureIndex.sol` a user can vote on behalf of others using signatures, the `batchVoteForManyWithSig()` is one of them however the way it is designed creates an issue, if one sognature in 100 other valid signatures is invalid it will cause the whole transaction to revert, this might cause the other signatures to expire considering the time it would take to identify and exclude the invalid ones, a better implementation would look like this
```js
    function batchVoteForManyWithSig(
        address[] memory from,
        uint256[][] calldata pieceIds,
        uint256[] memory deadline,
        uint8[] memory v,
        bytes32[] memory r,
        bytes32[] memory s
    ) external nonReentrant {
        uint256 len = from.length;
        require(
            len == pieceIds.length && len == deadline.length && len == v.length && len == r.length && len == s.length,
            "Array lengths must match"
        );

        for (uint256 i; i < len; i++) {
            if (_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i]));
                   _voteForMany(pieceIds[i], from[i]);
        }
    }
```