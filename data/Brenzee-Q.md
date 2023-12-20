# Low Severity issues

## `MaxHeap.insert` function is expected to revert if the heap is full, but that never happens.

[Link To Code](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L115-L130)

In `MaxHeap` contract [line 116](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L116) dev comment states:
```solidity
    /// @dev The function will revert if the heap is full
```

This indicates that the function is expected to revert when the heap is full. 
However, the function neither reverts when the heap is full nor checks if the heap is full.

[`MaxHeap.insert`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L119-L130)
```solidity
    function insert(uint256 itemId, uint256 value) public onlyAdmin {
        heap[size] = itemId;
        valueMapping[itemId] = value; // Update the value mapping
        positionMapping[itemId] = size; // Update the position mapping

        uint256 current = size;
        while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
            swap(current, parent(current));
            current = parent(current);
        }
        size++;
    }
```

If it is intended to revert when the heap is full, recommend to add a check for the heap size and revert if the heap is full. If not - recommend to remove the dev comment.

## If no one bids on an art piece, the art piece will be burned and all the votes for it are lost.

In `AuctionHouse` there can be an instance where the auction settles without any bids on an art piece. In this case, the art piece will be burned and all the votes for it are lost.

[`AuctionHouse._settleAuction`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L356-L359)

```solidity
    //If no one has bid, burn the Verb
    if (_auction.bidder == address(0))
        verbs.burn(_auction.verbId);
```

The issue is that if this happens, all of the votes for it will be gone since it is not added back into the `CultureIndex` and `MaxHeap` contracts. Meaning that creators lost all the votes for their art piece.

If that is the recommended behavior, no changes necessary.
If not, recommend to create a mechanism where the art piece is reset back in `CultureIndex` and `MaxHeap` contracts. In this way creators will not need to start from scratch.

## Incorrect checks in the context of the variable meaning

1. [`RewardsSplits.computeTotalReward`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40-L52)
`RewardsSplits.computeTotalReward` function checks if the `paymentAmountWei` is greater than `minPurchaseAmount` and less than `maxPurchaseAmount`. If not - it reverts.

```solidity
        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

According to this "if" statement - if the `paymentAmountWei` is equal to `minPurchaseAmount` or `maxPurchaseAmount`, the function will revert, which would be unexpected for a user who expects `minPurchaseAmount` or `maxPurchaseAmount` to be the minimum and maximum values respectively. 

Recommend to change the "if" statement to:
```solidity
        if (paymentAmountWei < minPurchaseAmount || paymentAmountWei > maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

2. [`CultureIndex._vote`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L307-L324)

For user to successfully vote for a art piece, the `CultureIndex._vote` function checks if the user's vote `weight` is greater than `minVoteWeight`
```solidity
        require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");
```

According to this "if" statement - if the `weight` is equal to `minVoteWeight`, the function will revert, which would be unexpected for a user who expects `minVoteWeight` to be the minimum value.

Recommend to change the "if" statement to:
```solidity
        require(weight >= minVoteWeight, "Weight must be greater than or equal to minVoteWeight");
```

## When `MaxHeap.extractMax` is called, `valueMapping` and `positionMapping` are not reset

[Link to code](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L156-L164)

When `MaxHeap.extractMax` is called, the max value of the heap is removed and the heap is restructured. However, the `valueMapping` and `positionMapping` are not reset. This means that the `valueMapping` and `positionMapping` will contain the old values of the removed item.

Recommend to reset the `valueMapping` and `positionMapping` when `MaxHeap.extractMax` is called.
```solidity
    function extractMax() external onlyAdmin returns (uint256, uint256) {
        require(size > 0, "Heap is empty");

        uint256 popped = heap[0];
        uint256 value = valueMapping[popped];
        delete valueMapping[popped];
        delete positionMapping[popped];
        heap[0] = heap[--size];
        maxHeapify(0);

        return (popped, value);
    }
```

