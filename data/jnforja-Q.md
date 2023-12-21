## `RewardSplits::computeTotalRewards` min and max limits should be inclusive and not exclusive for semantics' sake.

```diff
- if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
+ if (paymentAmountWei < minPurchaseAmount || paymentAmountWei > maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

## Lack of value validation for ` AuctionHouse::minBidIncrementPercentage`
In the same way that the values of `creatorRateBps`, `entropyRateBps`, `minCreatorRateBps` are validated whenever they're set, the same should be done for `minBidIncrementPercentage`. 

## `CultureIndex::createPiece` Natspec incorrectly specifies the requirements for `metadata` parameter

Natspec in `CultureIndex::createPiece` specifies `metadata` restriction as follows
```
`metadata` must include name, description, and image. Animation URL is optional.
```
However, the description isn't correct as it doesn't match how `CultureIndex::validateMediaType` works.

## `CultureIndex::_setQuorumVotesBPS` Natspec specifies wrong restrictions for `newQuorumVotesBPS` value

`CultureIndex::_setQuorumVotesBPS` Natspec says 
```
@dev newQuorumVotesBPS must be greater than the hardcoded min
```
However, the code defines that newQuorumVotesBPS should be smaller than hardcoded max

```solidity
require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");
```
