# Redundant check

## Related  code:
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L486-L491
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L169-L172
## Proof of Concept
In `MaxHeap:: getMax` , it will always check size > 0, there is no need to check it at `CultureIndex::topVotedPieceId`

```
    function topVotedPieceId() public view returns (uint256) {
        // @audit reduant check
        require(maxHeap.size() > 0, "Culture index is empty");
        //slither-disable-next-line unused-return
        (uint256 pieceId, ) = maxHeap.getMax();
        return pieceId;
    }

```

```

    function getMax() public view returns (uint256, uint256) {
        require(size > 0, "Heap is empty");
        return (heap[0], valueMapping[heap[0]]);
    }


```