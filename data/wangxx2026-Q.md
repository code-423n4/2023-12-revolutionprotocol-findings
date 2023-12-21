## Lack of necessary parameter checks

[setMinBidIncrementPercentage](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L297-L301) There is no check on minBidIncrementPercentage left parameter, if minBidIncrementPercentage is 0, the bid can be successful without increasing.

[setReservePrice](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L287-L291) No parameter check on reservePrice, if reservePrice is 0, you can get the bidding for free.