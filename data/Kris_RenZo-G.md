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


&nbsp;
&nbsp;
&nbsp;

# Convert Non-State Accessing Functions to Pure to Save Gas

## Summary

The following functions can be restricted to pure functions: `transfer(); _transfer(); transferFrom; approve; _approve; _approve; _spendAllowance`.

## Vulnerability Details

Due to the non-transferable nature of the token, functions related to transfer, such as `_transfer(); transferFrom; approve; _approve; _approve; _spendAllowance`, do not read or write to the contract storage. Consequently, marking these functions as pure or view can enhance gas efficiency.

## Impact

Saving gas costs for users.

## Tools Used

Manual review

## Recommended Mitigation Steps

Change the following functions to pure functions: `_transfer(); transferFrom; approve; _approve; _approve; _spendAllowance`.

&nbsp;



&nbsp;
&nbsp;
&nbsp;
# Gas-Inefficient `_mintTo()` Implementation

## Summary

The current implementation of `_mintTo()` is gas-inefficient, especially when dealing with multiple creators due to its reliance on storage variables.

## Vulnerability Details

Optimizing the `_mintTo()` function involves using a `memory` variable (`newPiece`) instead of a `storage` variable for improved gas efficiency ([see here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L296C20-L296C20)). By creating a memory copy of `ArtPiece`, making necessary modifications, and then assigning it to the `artPieces` mapping in storage only once, the number of storage operations (costly in terms of gas) is reduced.

## Impact

Saves gas cost for users.

## Tools Used

Manual review

## Recommended Mitigation Steps

Below is a more gas-efficient implementation of the `createPiece()` function:

```solidity
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

        // Create a memory copy of the art piece
        ICultureIndex.ArtPiece memory newPiece = artPiece;

        // Perform all necessary modifications to newPiece in memory
        // ...

        // Assign the memory art piece to the storage mapping only once
        artPieces[verbId] = newPiece;

        _mint(to, verbId);

        emit VerbCreated(verbId, artPiece);

        return verbId;
    } catch {
        // Handle failure (e.g., revert, emit an event, set a flag, etc.)
        revert("dropTopVotedPiece failed");
    }
}
```
