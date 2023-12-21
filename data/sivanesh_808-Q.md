

| S.No | Title |
|------|-------|
| L-01 | Rounding Errors in Reward Calculation in `RewardsSplits.sol` |
| L-02 | Incomplete Reward Distribution in `RewardsSplits.sol` |
| L-03 | Lack of Validation in `updateValue` Function in `MaxHeap.sol` |
| L-04 | Potential Inefficiency in `maxHeapify` Function in `MaxHeap.sol` |
| L-05 | Flawed Balance Check in `extractMax` Function in `MaxHeap.sol` |
| L-06 | Potential Miscalculation in `maxHeapify` Downwards Heapify Logic in `MaxHeap.sol` |



### [L-01] Rounding Errors in Reward Calculation.

### File : RewardsSplits.sol

### Description:
The contract's `computeTotalReward` and `computePurchaseRewards` functions are responsible for calculating various rewards based on a payment amount. These calculations use integer division, which can lead to rounding errors. In Solidity, integer division truncates towards zero, potentially causing minor but notable discrepancies, particularly with small payment amounts.

### Code Snippet:
```solidity
    function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

        return
            (paymentAmountWei * BUILDER_REWARD_BPS) /
            10_000 +
            (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
            10_000 +
            (paymentAmountWei * DEPLOYER_REWARD_BPS) /
            10_000 +
            (paymentAmountWei * REVOLUTION_REWARD_BPS) /
            10_000;
    }

    function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
        return (
            RewardsSettings({
                builderReferralReward: (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000,
                purchaseReferralReward: (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000,
                deployerReward: (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000,
                revolutionReward: (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000
            }),
            computeTotalReward(paymentAmountWei)
        );
    }
```

### Example:
Suppose a user makes a payment of `1 wei`. The rewards are calculated as a percentage of this amount:

- Builder Referral Reward (BUILDER_REWARD_BPS = 100): `1 wei * 100 / 10,000 = 0 wei` (rounded down from `0.01 wei`)
- Purchase Referral Reward (PURCHASE_REFERRAL_BPS = 50): `1 wei * 50 / 10,000 = 0 wei` (rounded down from `0.005 wei`)

As seen in these examples, the actual reward amounts get rounded down to `0 wei`, leading to a loss of rewards due to rounding.

### Expected Behavior:
The reward calculations should handle small amounts more accurately, minimizing the impact of rounding errors.

### Actual Behavior:
Due to integer division, reward calculations for small payment amounts are heavily affected by rounding down, leading to potential loss of rewards.

### Updated Code:
Adjust the calculation method to reduce the impact of rounding errors. One approach is to aggregate all rewards first and then divide:

```solidity
function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
    uint256 totalBPS = BUILDER_REWARD_BPS + PURCHASE_REFERRAL_BPS + DEPLOYER_REWARD_BPS + REVOLUTION_REWARD_BPS;
    return (paymentAmountWei * totalBPS) / 10_000;
}

function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
    uint256 scaledAmount = paymentAmountWei * 10_000;
    return (
        RewardsSettings({
            builderReferralReward: (scaledAmount * BUILDER_REWARD_BPS) / 10_000 / 10_000,
            purchaseReferralReward: (scaledAmount * PURCHASE_REFERRAL_BPS) / 10_000 / 10_000,
            deployerReward: (scaledAmount * DEPLOYER_REWARD_BPS) / 10_000 / 10_000,
            revolutionReward: (scaledAmount * REVOLUTION_REWARD_BPS) / 10_000 / 10_000
        }),
        computeTotalReward(paymentAmountWei)
    );
}
```

### Impact:
The rounding errors, while minor per transaction, can accumulate over time, especially in a system with a high volume of transactions involving small amounts. Accurate reward distribution is vital for maintaining fairness and user trust in the system.

### Github : [RewardsSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40)

---------------------------------------------------------------------------------------------------------------------------


### [L-02]: Incomplete Reward Distribution Due to Non-Complementary Basis Points

### File : RewardsSplits.sol

### Description:
The contract's reward calculation mechanism is based on fixed basis points that do not sum up to 100% (10,000 BPS). This results in a portion of the total payment amount not being distributed as rewards, leading to an incomplete reward distribution.

### Code Snippet:
```solidity
uint256 internal constant DEPLOYER_REWARD_BPS = 25;
uint256 internal constant REVOLUTION_REWARD_BPS = 75;
uint256 internal constant BUILDER_REWARD_BPS = 100;
uint256 internal constant PURCHASE_REFERRAL_BPS = 50;

function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
    return
        (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000 +
        (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000 +
        (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000 +
        (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000;
}
```

### Detailed Calculation Example:
For a payment amount of `1000 wei`, the rewards are calculated as follows:

1. **Builder Referral Reward**:
   - BPS: 100
   - Calculation: `(1000 wei * 100) / 10,000 = 10 wei`

2. **Purchase Referral Reward**:
   - BPS: 50
   - Calculation: `(1000 wei * 50) / 10,000 = 5 wei`

3. **Deployer Reward**:
   - BPS: 25
   - Calculation: `(1000 wei * 25) / 10,000 = 2.5 wei` (rounded down to `2 wei` in Solidity)

4. **Revolution Reward**:
   - BPS: 75
   - Calculation: `(1000 wei * 75) / 10,000 = 7.5 wei` (rounded down to `7 wei` in Solidity)

Totaling these up, we get `10 wei + 5 wei + 2 wei + 7 wei = 24 wei`. However, the original payment was `1000 wei`.

The sum of the BPS values is `100 + 50 + 25 + 75 = 250`. Since this sum does not equal 10,000 (100%), there's a discrepancy. The total BPS should account for the entire payment, but in this case, it only accounts for 2.5% of the payment, leaving 97.5% of the payment amount undistributed.

### Impact:
The impact of this bug is significant, especially for large payment amounts. A considerable portion of the payment is not being distributed, which could either accumulate as excess funds in the contract or represent a loss of funds for parties entitled to rewards. This discrepancy can have financial implications and may also affect the perceived fairness and credibility of the reward distribution mechanism.

### Github : [RewardsSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L54)


----------------------------------------------------------------------------------------------------------------------------
### [L-03] Lack of Validation in `updateValue` Function

### File : MaxHeap.sol

### Description:
The `updateValue` function in the contract is designed to update the value of an existing item in the heap. However, there's no validation to check whether the `itemId` provided as an argument actually exists in the heap. This omission could lead to unintended behavior if an `itemId` that does not exist in the heap is passed to the function.

### Code Snippet:
```solidity
function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
    uint256 position = positionMapping[itemId];
    uint256 oldValue = valueMapping[itemId];
    // Update the value in the valueMapping
    valueMapping[itemId] = newValue;
    // ... rest of the function ...
}
```

### Expected Behavior:
The function should validate that the provided `itemId` exists in the heap before proceeding with the update.

### Actual Behavior:
There is no check to ensure the `itemId` is a valid and existing item in the heap, potentially leading to updates on non-existent items.

### Updated Code:
Add a validation check to confirm the existence of `itemId` in the heap.

```solidity
function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
    require(positionMapping[itemId] < size, "Item does not exist in the heap");
    uint256 position = positionMapping[itemId];
    uint256 oldValue = valueMapping[itemId];
    // Update the value in the valueMapping
    valueMapping[itemId] = newValue;
    // ... rest of the function ...
}
```

### Impact:
Without this validation, the function could incorrectly update the `valueMapping` and `positionMapping` for an `itemId` that doesn't exist in the heap. This could lead to inconsistencies in the heap's data and potentially affect the integrity of the heap structure.

### Github : [MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L136)

---------------------------------------------------------------------------------------------------------------------



### [L-04] Potential Inefficiency in `maxHeapify` Function

### File : MaxHeap.sol

### Description:
The `maxHeapify` function is intended to maintain the max-heap property in the heap data structure. However, the implementation appears to be inefficient in certain scenarios, specifically when the given position `pos` does not need to be heapified. The function currently proceeds with comparisons and potential swaps even if the node at `pos` already satisfies the max-heap property.

### Code Snippet:
```solidity
function maxHeapify(uint256 pos) internal {
    uint256 left = 2 * pos + 1;
    uint256 right = 2 * pos + 2;
    uint256 posValue = valueMapping[heap[pos]];
    uint256 leftValue = valueMapping[heap[left]];
    uint256 rightValue = valueMapping[heap[right]];

    if (pos >= (size / 2) && pos <= size) return;

    if (posValue < leftValue || posValue < rightValue) {
        // ... swapping logic ...
    }
}
```

### Expected Behavior:
The `maxHeapify` function should quickly determine if the node at `pos` is already in the correct position to maintain the max-heap property and avoid unnecessary comparisons and swaps.

### Actual Behavior:
The function does not efficiently check if the heap property is already satisfied for the node at `pos`. It proceeds with computing the left and right values even if it's not necessary, leading to potential inefficiencies.

### Updated Code:
Add a more efficient check at the beginning of the function to determine if heapifying is needed.

```solidity
function maxHeapify(uint256 pos) internal {
    if (pos >= size) return; // Early return if pos is out of bounds

    uint256 largest = pos;
    uint256 left = 2 * pos + 1;
    uint256 right = 2 * pos + 2;

    if (left < size && valueMapping[heap[left]] > valueMapping[heap[largest]]) {
        largest = left;
    }
    if (right < size && valueMapping[heap[right]] > valueMapping[heap[largest]]) {
        largest = right;
    }

    if (largest != pos) {
        swap(pos, largest);
        maxHeapify(largest);
    }
}
```

### Impact:
While not a critical bug, optimizing the `maxHeapify` function can reduce the number of operations performed, especially in scenarios where this function is called frequently. This change can help in maintaining the efficiency of the heap operations, ensuring that the contract's logic is not only correct but also optimized for performance in terms of operation counts.

### Github : [MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L94)

------------------------------------------------------------------------------------------------------------

### [L-05] Flawed Balance Check in `extractMax` Function

### File : MaxHeap.sol

### Description:
The `extractMax` function is intended to remove the maximum element from the heap. However, there appears to be a logical issue with how the balance of the heap is maintained after the extraction. The function does not adequately handle the reassignment and rebalancing of the heap after the maximum element is removed, which can lead to an unbalanced heap.

### Code Snippet:
```solidity
function extractMax() external onlyAdmin returns (uint256, uint256) {
    require(size > 0, "Heap is empty");

    uint256 popped = heap[0];
    heap[0] = heap[--size];
    maxHeapify(0);

    return (popped, valueMapping[popped]);
}
```

### Expected Behavior:
After the maximum element is removed, the heap should be properly rebalanced to maintain its max-heap property. This involves appropriately rearranging the remaining elements.

### Actual Behavior:
The current implementation reduces the size of the heap and attempts to rebalance it using `maxHeapify(0)`. However, this approach might not correctly rebalance the heap if the element moved to the root (heap[0]) is not the next highest value, potentially leading to an incorrect heap structure.

### Updated Code:
An improved approach would involve checking and potentially rearranging the heap more thoroughly to ensure the max-heap property is maintained.

```solidity
function extractMax() external onlyAdmin returns (uint256, uint256) {
    require(size > 0, "Heap is empty");

    uint256 popped = heap[0];
    uint256 poppedValue = valueMapping[popped];
    delete positionMapping[popped];
    delete valueMapping[popped];

    if (size > 1) {
        heap[0] = heap[size - 1];
        positionMapping[heap[0]] = 0;
        maxHeapify(0);
    }
    size--;

    return (popped, poppedValue);
}
```

### Impact:
If the heap is not correctly rebalanced after an extraction, it can violate the max-heap property, leading to incorrect behavior in subsequent heap operations. This could impact the integrity and reliability of the data structure, affecting the overall logic and functionality of the contract.

### Github : [MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L156)
----------------------------------------------------------------------------------------------------------------------------------



### [L-06] Potential Miscalculation in `maxHeapify` Downwards Heapify Logic.

### File : MaxHeap.sol

### Description:
The `maxHeapify` function is designed to maintain the max-heap property by appropriately adjusting the heap structure. However, there is a potential miscalculation in the logic that decides whether to perform upwards or downwards heapify. The function assumes that if the new value is less than the old value, it should always perform downwards heapify. This assumption might not always hold true, especially in edge cases where the updated value is still greater than its children but less than its original value.

### Code Snippet:
```solidity
function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
    // ... other code ...
    if (newValue > oldValue) {
        // Upwards heapify
    } else if (newValue < oldValue) {
        maxHeapify(position); // Downwards heapify
    }
}
```

### Expected Behavior:
The logic should correctly determine whether to perform upwards or downwards heapify based on the new value's position in the heap and its comparison with both parent and children nodes.

### Actual Behavior:
The function only checks if the new value is less than the old value to decide on downwards heapify, which might not be sufficient for maintaining the max-heap property in certain scenarios.

### Updated Code:
Enhance the logic to consider both parent and children nodes for a more accurate decision on heapify direction:

```solidity
function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
    uint256 position = positionMapping[itemId];
    uint256 oldValue = valueMapping[itemId];
    valueMapping[itemId] = newValue;

    if (position != 0 && newValue > valueMapping[heap[parent(position)]]) {
        // Upwards heapify
        while (position != 0 && newValue > valueMapping[heap[parent(position)]]) {
            swap(position, parent(position));
            position = parent(position);
        }
    } else {
        maxHeapify(position); // Downwards heapify
    }
}
```

### Impact:
Failing to correctly determine the appropriate heapify direction can lead to a violation of the max-heap property. This could result in the heap structure not accurately representing the intended priority order, potentially affecting the contract's logic that relies on the heap for decision-making or processing.

### Github : [MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L136)

----------------------------------------------------------------------------------------------------------------------------------

