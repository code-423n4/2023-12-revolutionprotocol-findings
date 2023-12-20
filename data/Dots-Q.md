# Bidder is able to bid as many times as he wants while being the `auction.bidder`

## Impact
Low. 

## Summary
The bidder could potentially indefinitely stall the auction from settling by outbidding himself.

## Proof of Concept
There is no check if `msg.sender == auction.bidder` in `function createBid` (AuctionHouse.sol). So the bidder could potentially indefinitely stall the auction from settling by outbidding himself.

## Tools Used
manual review

## Recommended Mitigation Steps
add a check for `msg.sender == auction.bidder`