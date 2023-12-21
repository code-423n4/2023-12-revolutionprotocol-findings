# Gas-Inefficient `createPiece()` Implementation

## Summary

The current implementation of `createPiece()` is gas-inefficient, especially when dealing with multiple creators due to its reliance on storage variables.

&nbsp;

## Vulnerability Details

Optimizing the `createPiece()` function involves using a `memory` variable (`newPiece`) instead of a `storage` variable for improved gas efficiency ([see here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L223)). By creating a memory copy of `newPiece`, making necessary modifications, and then assigning it to the `pieces` mapping in storage only once, the number of storage operations (costly in terms of gas) is reduced.

&nbsp;

## Impact

Saves gas cost for users.

&nbsp;

## Tools Used

Manual review

&nbsp;

## Recommended Mitigation Steps

Below is a more gas-efficient implementation of the `createPiece()` function:

```solidity
function createPiece(
    ArtPieceMetadata calldata metadata,
    CreatorBps[] calldata creatorArray
) public returns (uint256) {
    uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

    // Validate the media type and associated data
    validateMediaType(metadata);

    uint256 pieceId = _currentPieceId++;

    // Create a new in-memory ArtPiece struct
    ArtPiece memory newPiece;

    newPiece.pieceId = pieceId;
    newPiece.totalVotesSupply = _calculateVoteWeight(
        erc20VotingToken.totalSupply(),
        erc721VotingToken.totalSupply()
    );
    newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
    newPiece.metadata = metadata;
    newPiece.sponsor = msg.sender;
    newPiece.creationBlock = block.number;
    newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;

    // Copy creators from calldata to memory
    newPiece.creators = new CreatorBps[](creatorArrayLength);
    for (uint i; i < creatorArrayLength; i++) {
        newPiece.creators[i] = creatorArray[i];
    }

    // Insert the new piece into the max heap
    maxHeap.insert(pieceId, 0);

    // Assign the in-memory ArtPiece struct to storage
    pieces[pieceId] = newPiece;

    emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

    // Emit an event for each creator
    for (uint i; i < creatorArrayLength; i++) {
        emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
    }

    return pieceId;
}
```

This optimized implementation reduces gas costs by minimizing storage operations and enhances the overall efficiency of the `createPiece()` function.
