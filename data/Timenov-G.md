## Summary
[G-1] Combine logic in same `if` statements.
[G-2] Assigning to the same value.

### [G-1] Combine logic in same `if` statements.
In `AuctionHouse::createBid` we have `if (extended)` twice. The first we change the `auction.endTime`, in the second we emit an event. Consider using one check that does both things.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L192

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L199

### [G-2] Assigning to the same value.
In `VerbsToken:_mintTo` first gets `artPiece` from `cultureIndex.getTopVotedPiece()`. Later calls `cultureIndex.dropTopVotedPiece()` and assigns the returns value to `_artPiece`. The next line is `artPiece = _artPiece;`. If we look at `dropTopVotedPiece()`, we can see that it gets the piece from `getTopVotedPiece()`. So basically they are the same and this step can be skipped.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L282

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L292-L293