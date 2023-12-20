# Bidder is able to bid as many times as he wants while being the `auction.bidder`

## Impact
Low. 
The bidder could potentially indefinitely stall the auction from settling by out bidding himself.

## Proof of Concept
There is no check if `msg.sender == auction.bidder`. So the bidder could potentially indefinitely stall the auction from settling by out bidding himself.

## Tools Used
manual review

## Recommended Mitigation Steps
add a check for `msg.sender == auction.bidder`