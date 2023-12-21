### Reserved price can be altered while an auction is being settled

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L348

```
if (address(this).balance < reservePrice)
```

The line above is used to check if a contract's balance is greater than the reserve price before settling an auction, but it is possible to change the reservePrice variable before the auction is settled therefore bids that should be valid can be rendered invalid at auction settlement. Consider making the setReservePrice function only callable when an auction is unpaused i.e. when bids are still being made to prevent valid user bids from being invalidated.