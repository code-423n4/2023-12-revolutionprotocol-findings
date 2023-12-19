# Revolution Protocol Analysis by Timenov

## Summary
- Codebase Analysis
- Potential security concerns
- Approach used to review
- What can be learned

### Codebase Analysis
I have spent `~15 hours` reviewed the codebase. Here is what I have found:

In the following diagram we can examine how the protocol works and how users can interact with it.

![ActionDiagram](https://i.imgur.com/xcDlOrT.png)

1. `Creator(s)` add their `art` in the `CultureIndex.sol`.
    - It can be of type `IMAGE`, `ANIMATION` or `TEXT`.
2. After `art` is added, `voters` can vote for it using their `ERC20` or `ERC721` tokens.
3. Every day `VerbsToken.sol` calls `CultureIndex.sol` to get the most voted `art`.
    - `Arts` are stored in `MaxHeap.sol`
4. `VerbsToken.sol` mints `VerbToken` to `AuctionHouse.sol` with the same `metadata` as the winning `art`.
5. `AuctionHouse.sol` creates an auction and `bidders` can bid for it.
6. Highest bidder gets the `art`, `creators` get a portion of the bid and `ERC20` tokens and the `owner` gets the remaining `ETH` from the bid. 
    - If the contract can't pay the `creator(s)`, the `bidder` is refunded and the `verb` is burned.
    - If there is no `bidder`, the `verb` is burned.
 
### Potential security concerns
1. Test coverage is low (88%).
2. Functions have a lot of loops and external calls that could cause DoS.
3. State variables for other contracts don't have backup `setter` if set to wrong address by mistake.

#### Test coverage is low (88%).
Test coverage `MUST` be `100%` and cover all edge cases.

#### Functions have a lot of loops and external calls that could cause DoS.
There a few functions that have way much operations in them and can cause `DoS`. One of them is `AuctionHouse::settleCurrentAndCreateNewAuction`. First it calls `_settleAuction` which has multiple checks, `call` to send `ETH` to the owner, a `for` loop that sends `ETH` to all creators of the `art` and mints ERC20. Yes, every call has a gas limit of `50_000`, but still the protocol allows the creators to be up to `100`. After that it calls `_createAuction` which even has a check if the `gasLeft >= 750_000` and can fail there.

#### State variables for other contracts don't have backup `setter` if set to wrong address by mistake.
All contracts have state variables that are references to other contracts. However there can be only set in the `initialize` function. It is good to have a backup fucntion that allows these references to be changed if set to the wrong addresses.

### Approach used to review
The approach I used to review the codebase is the following:
1. Read all the documentation about the protocol.
2. Read all findings of the Nouns DAO.
3. Briefly read the code.
4. Went back to the documentation and drew diagrams.
5. Deep dived into the code and searched for attacked vectors.

### What can be learned
The code is well structured. NatSpec explains the variables and functions. All contracts are packed and work together very well.

### Time spent:
15 hours