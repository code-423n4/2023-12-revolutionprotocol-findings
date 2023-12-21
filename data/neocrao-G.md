## [1] TokenEmitterRewards::_handleRewardsAndGetValueToSend() can be optimized to save gas

`TokenEmitterRewards::_handleRewardsAndGetValueToSend()` inherits from `RewardSplits.sol`, and many methods from `RewardSplits.sol` are used in `TokenEmitterRewards::_handleRewardsAndGetValueToSend()` itself.

Code references:
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18-L20

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L54

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L66

Below is the flow roughly:

```solidity
* TokenEmitterRewards::_handleRewardsAndGetValueToSend() calls `RewardSplits::computeTotalReward()`
* TokenEmitterRewards::_handleRewardsAndGetValueToSend() calls `RewardSplits::_depositPurchaseRewards()`
    * TokenEmitterRewards::_depositPurchaseRewards() calls `RewardSplits::computePurchaseRewards()`
    * `RewardSplits::computePurchaseRewards()` calls `RewardSplits::computeTotalReward()`
```

We can see that `RewardSplits::computeTotalReward()` is called twice.

The following refactor can save gas:

```diff
// SPDX-License-Identifier: MIT
pragma solidity 0.8.22;

import { RewardSplits } from "../RewardSplits.sol";

abstract contract TokenEmitterRewards is RewardSplits {

    ...
    ...

    function _handleRewardsAndGetValueToSend(
        uint256 msgValue,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {
+       (RewardsSettings memory settings, uint256 totalReward) = computePurchaseRewards(msgValue);

        if (msgValue < totalReward) revert INVALID_ETH_AMOUNT();

-       return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
+       return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer, totalReward, settings);
    }
}

abstract contract RewardSplits {

    ...
    ...

    function _depositPurchaseRewards(
        uint256 paymentAmountWei,
        address builderReferral,
        address purchaseReferral,
-       address deployer
+       address deployer,
+       uint256 totalReward,
+       RewardsSettings memory settings
    ) internal returns (uint256) {
-       (RewardsSettings memory settings, uint256 totalReward) = computePurchaseRewards(paymentAmountWei);

        if (builderReferral == address(0)) builderReferral = revolutionRewardRecipient;

        ...
        ...
    }
}
```

The `TokenEmitterRewards::_handleRewardsAndGetValueToSend()` is used by `ERC20TokenEmitter::buyToken()`, and following are the gas savings:

```
Original:
| Function Name                                        | min  | avg    | median | max
| buyToken                                             | 8086 | 177201 | 253198 | 358233

After update:
| Function Name                                        | min  | avg    | median | max
| buyToken                                             | 8086 | 176739 | 252665 | 357700

Savings:
| Function Name                                        | min  | avg    | median | max
| buyToken                                             | 0    | 462    | 533    | 533
```

## [2] Fetch the art piece only once in AuctionHouse::_settleAuction() to save gas

Code Reference:
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L336

The `AuctionHouse::_settleAuction()` function calls `verbs.getArtPieceById(_auction.verbId)` multiple times.

```solidity
    function _settleAuction() internal {
        ...
        ...

                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

                ...
                ...
                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
    }
```

These calls can be cached, as the same art piece is retrieved multiple times.

The code can be updated as follows to avoid making multiple external calls:

```diff
    function _settleAuction() internal {
        ...
        ...

+       ICultureIndex.ArtPiece memory artPiece = verbs.getArtPieceById(_auction.verbId);

        ...
        ...

-               uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
-               address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
+               uint256 numCreators = artPiece.creators.length;
+               address deployer = artPiece.sponsor;

                ...
                ...
-                       ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
+                       ICultureIndex.CreatorBps memory creator = artPiece.creators[i];
    }
```