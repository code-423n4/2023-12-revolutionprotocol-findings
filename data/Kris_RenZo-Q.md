# Typographical Error in Code Comments

## Summary

The AuctionHouse contract incorrectly refers to its ERC721 token by a different name.

&nbsp;

## Vulnerability Details

In [line 354](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L354C33-L354C33) of the AuctionHouse contract, the comments mistakenly mention the ERC721 token to burn as "Noun" instead of "Verb." The correct reference should be "Verb," as this contract was forked from the original Noun contract.

&nbsp;

## Impact

This typo can mislead and confuse reviewers of the contract.

&nbsp;

## Tools Used

Manual review

&nbsp;

## Recommended Mitigation Steps

Correct the typo by changing "Noun" to "Verb" in the comments.


&nbsp;
&nbsp;
&nbsp;
# Lack of Event Emission After Sensitive Actions

## Summary

Best practices recommend emitting events after every state change to enhance transparency and off-chain tracking.

&nbsp;

## Vulnerability Details

The `AuctionHouse::initialized()` function lacks event emissions for the following state changes:

- `timeBuffer = _auctionParams.timeBuffer`
- `reservePrice = _auctionParams.reservePrice`
- `minBidIncrementPercentage = _auctionParams.minBidIncrementPercentage`
- `duration = _auctionParams.duration`
- `creatorRateBps = _auctionParams.creatorRateBps`
- `entropyRateBps = _auctionParams.entropyRateBps`

&nbsp;

## Impact

Poor Developer Experience: The absence of events or poorly designed events makes it challenging for blockchain applications and tools that monitor blockchain activities to keep track of the actions performed within the AuctionHouse contract.

&nbsp;

## Tools Used

Manual review

&nbsp;

## Recommended Mitigation Steps

It is strongly recommended to emit events after every state change and to index event parameters of type address to improve transparency and off-chain tracking. Consider adding events for each of the mentioned state changes in the `AuctionHouse::initialized()` function.
