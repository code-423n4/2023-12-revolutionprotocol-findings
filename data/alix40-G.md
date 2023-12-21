# Gas Optimization Report
## Finding [G-01]: MaxHeap.sol - maxHeapify Function

### Description:
In the maxHeapify function of MaxHeap.sol, the check if (pos >= (size / 2) && pos <= size) return; is currently placed towards the end of the function. It is suggested to move this check to the beginning of the function to potentially save gas. By doing so, the unnecessary initialization of variables in the last iteration of maxHeapify can be avoided.

### Current Implementation:
```solidity

function maxHeapify(uint256 pos) internal {
    uint256 left = 2 * pos + 1;
    uint256 right = 2 * pos + 2;

    uint256 posValue = valueMapping[heap[pos]];
    uint256 leftValue = valueMapping[heap[left]];
    uint256 rightValue = valueMapping[heap[right]];

    if (pos >= (size / 2) && pos <= size) return;

    // Rest of the function...
}
```
### Suggested Implementation:

```solidity

function maxHeapify(uint256 pos) internal {
    if (pos >= (size / 2) && pos <= size) return;

    uint256 left = 2 * pos + 1;
    uint256 right = 2 * pos + 2;

    uint256 posValue = valueMapping[heap[pos]];
    uint256 leftValue = valueMapping[heap[left]];
    uint256 rightValue = valueMapping[heap[right]];

    // Rest of the function...
}
```
[Link to Code](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94C4-L94C4)
## Gas Optimization Finding [G-02]: Vote Storage Simplification

### Description:
In the current implementation, votes are stored in a struct called Vote which contains both the voterAddress and the weight. The votes are stored in the mapping mapping(uint256 => mapping(address => Vote)), where a piece ID is mapped to the voter's address, which is then further mapped to the Vote struct.

It is suggested to optimize gas usage by storing only the weight in the mapping and checking for the existence of a vote using votes[pieceId][voter] != 0 instead of votes[pieceId][voter].voterAddress != address(0).

### Current Implementation:

```solidity

struct Vote {
    address voterAddress;
    uint256 weight;
}

mapping(uint256 => mapping(address => Vote)) votes;

function hasVoted(uint256 pieceId, address voter) external view returns (bool) {
    return votes[pieceId][voter].voterAddress != address(0);
}
```
### Suggested Implementation:

```solidity

mapping(uint256 => mapping(address => uint256)) votes;

function hasVoted(uint256 pieceId, address voter) external view returns (bool) {
    return votes[pieceId][voter] != 0;
}
```
## Finding [G-03]: Redundant Check in _verifyVoteSignature

### Description:
In the _verifyVoteSignature function of CultureIndex.sol, there is a redundant check for address(0) within the ecrecover block. The from address is already validated for being non-zero earlier in the function. Therefore, the additional check for recoveredAddress == address(0) is redundant and can be safely removed.

The suggested implementation avoids the need to access the voterAddress within the Vote struct, potentially saving gas by simplifying the storage structure.
### Current Implementation:

```solidity

// ...

// Ensure to address is not 0
if (from == address(0)) revert ADDRESS_ZERO();

// Ensure signature is valid
if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

// ...
```
### Suggested Implementation:

```solidity

// ...

// Ensure signature is valid
if (recoveredAddress != from) revert INVALID_SIGNATURE();

// ...
```

[Link to Code](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L438C9-L442C1)
## Finding [G-04]: Simplification of Duplicate Vote Check

### Description:
In the _vote function of CultureIndex.sol, the check for duplicate votes currently uses require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");. This condition can be simplified for better gas efficiency to require(votes[pieceId][voter].voterAddress == address(0), "Already voted");.

### Current Implementation:

```solidity

// ...

require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");

// ...

### Suggested Implementation:
```
```solidity

// ...

require(votes[pieceId][voter].voterAddress == address(0), "Already voted");

// ...
```
[Link to Code](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L311)
## Finding [G-05]: Caching Verb Information in settleAuction()

### Description:
In the _settleAuction method of AuctionHouse.sol, the value of verbs.getArtPieceById(_auction.verbId) is called multiple times. Caching this value can save gas.

### Current Implementation:

```solidity

// ...

uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

// ...
```
### Suggested Implementation:

```solidity

// ...

IAuctionHouse.ArtPiece memory artPiece = verbs.getArtPieceById(_auction.verbId);  // Cache the result
uint256 numCreators = artPiece.creators.length;
address deployer = artPiece.sponsor;

// ...
```
[Link to Code](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L336C5-L414C6)