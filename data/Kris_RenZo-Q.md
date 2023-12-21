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

&nbsp;
&nbsp;
&nbsp;
# Initialization Timeframe Delay Causing Creators to Miss Rewards

## Summary

The `ERC20TokenEmitter` contract experiences an initialization timeframe delay, resulting in a period where creators receive zero rewards for all tokens bought.

&nbsp;

## Vulnerability Details

The variables `creatorRateBps` and `setEntropyRateBps` are not set during contract initialization, implying their values default to zero. This leads to zero tax being paid to creators (`creatorDirectPayment` and `totalTokensForCreators`) when users buy tokens. If this is intended, it should be clearly documented in the project's documentation. Otherwise, the current implementation deprives creators of their rewards.

&nbsp;

## Impact

Creators may feel frustrated for missing out on potential rewards, particularly if the amounts are significant.

&nbsp;

## Tools Used

Manual review

&nbsp;

## Recommended Mitigation Steps

Set the `creatorRateBps` and `setEntropyRateBps` variables in the `initialize()` function if the protocol intends creators to benefit from token buys from the start. Alternatively, clearly document these mechanics in the project's documentation if the delay in rewarding creators is intentional.
