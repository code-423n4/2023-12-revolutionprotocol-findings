# [G-01] MaxHeap::insert is only callable by CultureIndex::createPiece where insert::value will always be 0 so while loop will never run

**Description:**
The `value` passed into the `MaxHeap::insert` will always be 0, so the check `valueMapping[heap[current]] > valueMapping[heap[parent(current)]]` will never be true.

**Impact:**
As the first check of the while loop will always be false this check will be an unnecessary waste of gas and the code within the while loop is effectively dead so can be removed.

**Proof of Concept:**
`valueMapping[heap[current]]`
On the first pass through the while loop `heap[current]` is the most recently added item (`MOST_RECENTLY_ADDED`). `valueMapping[MOST_RECENTLY_ADDED]`is always zero. So the loop will fail.

**FINDING** Value mapping of most recently added index will always be 0.

**Recommended Mitigation:**
It's possible that the implementation of this max heap expects there to be non-zero values inserted into the heap so sorts the structure when each item is added. However that isn't the case here as all pieces start with zero votes. As the value to insert is always 0 it's safe to just add it onto the end of the heap as it will always be less than or equal to any other value in the heap.
