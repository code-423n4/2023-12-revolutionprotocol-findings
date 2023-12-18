# Title: Overbidden by the same bid amount
## Link to affected code
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L181

## Rating / Impact
Low severity rating, since this only applies in case of a zero or a low value in 'reservePrice" and if only very low bids are involved.

If the contract owner assigns a zero or a low value to the 'reservePrice,' and the auction receives very low bids, there is a potential risk of outbidding an existing bid with the 'same bid' amount. This is attributed to the presence of an equal sign, which, coupled with rounding issues in [L181](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L181), may result in an unintended overbid.

## Proof of Concept
See description in section observation.

## Tools Used
Manual review.

## Recommended Mitigation Steps
Consider changing code to 

`msg.value > _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100)`