## [G-01] `MaxHeap.maxHeapify()`: Early return should be placed at the top, before any storage reads.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L102

In the function `maxHeapify()`, there is an early breaking case:

```solidity
function maxHeapify(uint256 pos) internal {
  // ...
  if (pos >= (size / 2) && pos <= size) return; 
  // ...
}
```

The statement prevents any sort of processing when the node `pos` does not have exactly two children, i.e. there is nothing to heapify

Notice that there are **three** storage reads right before that. It turns out that `maxHeapify()` may hit the breaking case quite often, whenever a node is extracted through `extractMax()`. Then heapify may happen all the way to the lowest possible level of the heap, and we waste three storage reads whenever that happens.

Then it is strictly better to move the statement to the top of the function. Each redundant storage read costs **2100** gas, totalling to **6300** gas. Although the saving will not always happen, but gas cost cannot ever be larger with this optimization.

## [G-02] `MaxHeap`: Tightly pack storage (`size` variable) to save gas

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L67

The current storage layout of `maxHeap` is as following:

```solidity
address public admin; // 160 bits
mapping(uint256 => uint256) public heap;
uint256 public size = 0; // optimize this
```

Note that `size` can be packed alongside `admin` to fit in a single storage slot. 

The gas impact is as follow:
- Since every function is admin-only, `size` storage will always be read alongside `admin` in a function call. Then this optimization reduces one storage read, saving **2000** gas.
- Saves a further 17100 gas when `size` is set from zero to non-zero, as `admin` is already set. This is, however, a somewhat niche case that is applicable for the first art piece only.