|  | issues | instance |
|---|---|---|
| [L-01]| Consider using `nonReentrant`| 1 |

DETAILS

[L-01] Consider using `nonReentrant` Modifier
Although code already contains the `ReentrancyGuardUpgradeable` library from the OpenZeppelin Contracts, its best to use the `nonReentrant` modifier in the OpenZeppelin library. Both are designed to prevent reentrancy attacks, but while the `ReentrancyGuardUpgradeable` library prevents reentrancy at the contract level, the `nonReentrant` modifier is better because it prevents reentrancy at the function level,  This is a stronger approach to preventing reentrancy than the `ReentrancyGuardUpgradeable` library's approach, which uses the `beforeAcquire` function to check if the lock is already acquired.  The `nonReentrant` modifier uses a state variable to track whether a function is currently being executed. If the function is already being executed and an external call is made to it, the `nonReentrant` modifier will cause the function to revert, preventing reentrancy.The reentrancy vulnerability can be exploited in a function that performs state changes before making an external call.
```
solidity
/// @notice Extract the maximum element from the heap
/// @dev The function will revert if the heap is empty
/// @return The maximum element from the heap
function extractMax() external onlyAdmin returns (uint256, uint256) {
    require(size > 0, "Heap is empty");

    uint256 popped = heap[0];
    heap[0] = heap[--size];
    maxHeapify(0);

    return (popped, valueMapping[popped]);
}
```
In this function, the `maxHeapify(0)` function is called after the `heap[0]` value is reassigned. An attacker could potentially exploit this to perform a reentrancy attack. If the maxHeapify function makes an external call to another contract, an attacker could exploit this to perform a reentrancy attack. For example, if an attacker could call the insert function before the `maxHeapify` function completes, they could potentially insert a malicious value into the heap. This value could then be used to perform a reentrancy attack.

