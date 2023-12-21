## [1] The invalid ETH amount seems unnecessary

In `TokenEmitterRewards::_handleRewardsAndGetValueToSend()` function, it has the following check:

```
if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();
```

Code reference:
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18

`RewardSplits::computeTotalReward()` does the following math roughly (If we didnt have rounding issues):

Pseudocode:
```
totalBps = BUILDER_REWARD_BPS +
            PURCHASE_REFERRAL_BPS +
            DEPLOYER_REWARD_BPS +
            REVOLUTION_REWARD_BPS;

return paymentAmountWei * totalBps / 10_000
```

Now, the above can exceed `paymentAmountWei` only if `totalBps` is beyond `10_000`, that is, it is more than 100%, which is not possible, as the 4 BPS that make up the `totalBps` are constants and are way below 100%.

Hence, the following check can be removed from `TokenEmitterRewards::_handleRewardsAndGetValueToSend()` function

```
if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();
```

This will also provide some gas savings.

Alternatively, if check needs to be kept in place, then it will be more clear to revert with `INVALID_BPS_SUM()` error, as in this case its the BPS sum that exceeds `msgValue`, and not `msgValue` itself.

## [2] Missing checks for audio type

The function `CultureIndex::validateMediaType()` checks for the different types of media types. But it does not have any checks for audio media type.

In fact, the strict `ICultureIndex::ArtPieceMetadata` has no fields that might correspond specifically to audio, such as `audio url`, so this media type seems to only be partially supported. In such a case, the Audio media type should either be removed or disabled.

```solidity
struct ArtPieceMetadata {
    string name;
    string description;
    MediaType mediaType;
    string image;
    string text;
    string animationUrl;
}
```

## [3] AuctionHouse::createBid() has potential for rounding issues affecting smaller bids, where another bidder can win for the same bid

`AuctionHouse::createBid()` has the following require statement:

```
require(
    msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
    "Must send more than last bid by minBidIncrementPercentage amount"
);
```

Code references:
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L181

Now, this has a rounding issue if `(_auction.amount * minBidIncrementPercentage)` is less than `100`, as show below:

```
// Assume _auction.amount = 10
// Assume minBidIncrementPercentage = 9
//
=> msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100)
=> msg.value >= _auction.amount + ((10 * 9) / 100)
=> msg.value >= _auction.amount + (90 / 100)
=> msg.value >= _auction.amount + (0)
=> msg.value >= _auction.amount
```

So as long as `msg.value >= _auction.amount`, then the bid gets accepted.

So if a "User A" is the current bid winner with `10 gwei`, and if `minBidIncrementPercentage` is set to a value less than 10, then another user can come in with the same bid of `10 gwei` and take over the current winning bid position.

This might seem as not a big issue, as the bid amounts are very small, and this only works if `_auction.amount * minBidIncrementPercentage < 100`, which in the worst case would mean any value of `_auction.amount < 100`.

But this can cause inconvenience to the users, and non-awareness of this issue can lead to user's not realizing what happened, specially when some auction is being done with small amounts, and a user is winning the bid, but then they see that someone else won the bid for the same amount.

The issue is a Low issue, as the amount is very low, and also fortunately the following condition in `RewardSplits::computeTotalReward()` prevents such thing from happening:

```
if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

However, in the future if the contract is upgraded/updated and the `minPurchaseAmount` are changed, then this issue will start happening.

**Recommended Fix:**

Update the code as follows:

```solidity
    function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
        ...
        ...

+       require(
+          (_auction.amount * minBidIncrementPercentage) / 100 > 0
+           "Amount too low"
+       );

        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );

        ...
        ...
    }
```