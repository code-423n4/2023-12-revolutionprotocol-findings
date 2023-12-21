# QA Report: Solidity Smart Contract Audit

## Issue 1: Ineffective Use of `verbId` Parameter in `AuctionHouse.sol`

### Title
Ineffective Use of `verbId` Parameter in `AuctionHouse.sol`

### Impact
The issue in `AuctionHouse.sol` at line 174 results in the `verbId` parameter being redundant, as the function uses `_auction.verbId` for validation instead. This could lead to confusion, as the passed `verbId` parameter does not influence the function's behavior.

### Proof of Concept
The function `createBid` in `AuctionHouse.sol:174` is intended to create a bid for a specific Verb identified by `verbId`. However, the implementation validates `verbId` is the same as `auction.verbId`. This discrepancy renders the `verbId` parameter passed to the function useless.

### Tools Used
Manual Review

### Recommended Mitigation Steps
Consider either removing the `verbId` parameter from the function signature if it is indeed unnecessary, or modify the function logic to use the `verbId` parameter effectively.

### Issue Type
Invalid Validation

## Issue 2: Optimization Opportunity in `AuctionHouse.sol` for Gas Efficiency

### Title
Optimization Opportunity in `AuctionHouse.sol` for Gas Efficiency

### Impact
In `AuctionHouse.sol:374`, the repeated calls to `verbs.getArtPieceById(_auction.verbId)` result in unnecessary gas consumption due to multiple storage reads. This inefficiency can lead to increased transaction costs for users interacting with this function, particularly in the loop that follows.

### Proof of Concept
The issue is identified in the `AuctionHouse.sol` file at line 374. The code makes repeated calls to `verbs.getArtPieceById(_auction.verbId)`. This pattern is inefficient as it queries the same information multiple times from storage, a costly operation in terms of gas usage. Specifically, the `numCreators` and `deployer` variables are obtained through separate calls to the same function.

### Tools Used
Manual Review

### Recommended Mitigation Steps
To optimize gas usage, it is recommended to cache the result of `verbs.getArtPieceById(_auction.verbId)` in a local variable and use this variable for subsequent reads.

### Issue Type
Optimization
