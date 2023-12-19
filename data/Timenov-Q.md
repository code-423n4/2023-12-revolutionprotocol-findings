## Summary
[I-1] References can not be changed.
[I-2] Double `not`.
[L-1] Allow `extended` at `timeBuffer`.

### [I-1] References can not be changed.
In all contracts there are storage variables that are references to the other contracts. They are set in `initialize`. However there are no separete functions that allow these addresses to be changed. It is good practice to have these kind of functions with `onlyOwner` modifiers to allow changes if mistakes are made or other problems occur.

### [I-2] Double `not`.
It is bad to have double `not`. Change to logic to remove this. By double `not` I mean double `!`.

```diff
- require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
+ require(votes[pieceId][voter].voterAddress == address(0), "Already voted");
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L311

### [L-1] Allow `extended` at `timeBuffer`.
The protocol can extended the time of an `auction` if `_auction.endTime - block.timestamp < timeBuffer`. But if the `bid` is made just on the `timeBuffer`, the time should be extended as well.

```diff
- bool extended = _auction.endTime - block.timestamp < timeBuffer;
+ bool extended = _auction.endTime - block.timestamp <= timeBuffer;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L191