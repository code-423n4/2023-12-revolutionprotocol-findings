# [L-1] Using the maxHeap data structure leads to unfairness in art piece selection. 

The current implementation of maxHeap introduces a bias in art piece selection, as elements in the left branch are favored when two nodes have identical values. This issue arises during the maxHeapify process, where the left child is preferred over the right, regardless of their insertion order.

It means that the art piece in the left branch will always be priortize in selection if two art pieces have the same voting amount even though the left one is inserted later than the right one.

        function maxHeapify(uint256 pos) internal {
            uint256 left = 2 * pos + 1;
            uint256 right = 2 * pos + 2;

            uint256 posValue = valueMapping[heap[pos]];
            uint256 leftValue = valueMapping[heap[left]];
            uint256 rightValue = valueMapping[heap[right]];

            if (pos >= (size / 2) && pos <= size) return;

            if (posValue < leftValue || posValue < rightValue) {
                if (leftValue > rightValue) {
                    swap(pos, left);
                    maxHeapify(left);
                } else {
                    swap(pos, right);
                    maxHeapify(right);
                }
            }
        }

POC: Put the test in the MaxHeap test: packages/revolution/test/max-heap/MaxHeap.t.sol

```javascript
    function testLeftPriorityOverRight() public {
        maxHeap.insert(0, 17);
        maxHeap.insert(1, 11);
        maxHeap.insert(2, 16);
        maxHeap.insert(3, 12);
        maxHeap.insert(4, 13);
        maxHeap.insert(5, 16);
        (uint256 maxItemId, uint256 maxValue) = maxHeap.extractMax();
        console.log("maxItemId: ", maxItemId);
        console.log("maxValue: ", maxValue);
        // the item id 5 is the maximum over item id 2 even though item id 2 is inserted much earlier
        (maxItemId, maxValue) = maxHeap.getMax();
        console.log("maxItemId: ", maxItemId);
        console.log("maxValue: ", maxValue);
    }
```

## Recommended Mitigation Steps

Document this unexpected behavior for users so they can aware and does not feel unfair.

# [L-2] The hard-coded gas limit may not applicable in the future

The fixed gas limit of 50,000 in the _safeTransferETHWithFallback function lacks flexibility and may not align with future Ethereum network changes. A rigid gas limit can lead to failed transactions if the required gas exceeds this threshold due to change in Opcodes gas.

        assembly {
            // Transfer ETH to the recipient
            // Limit the call to 50,000 gas
            success := call(50000, _to, _amount, 0, 0, 0, 0)
        }

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/AuctionHouse.sol#L426-L430

## Recommended Mitigation Steps

Allow admin to update the gas limit.

# [L-3] The metadata input value is not sufficiently validated

When creating a new piece, the requirements for `metadata` are they must include name, description, and image. However, there is no input validation to make sure that the name, description and image are provided.

This oversight could result in incomplete or incorrect art piece registrations, as it currently doesn't enforce the presence of essential fields.

        function createPiece(
            ArtPieceMetadata calldata metadata,
            CreatorBps[] calldata creatorArray
        ) public returns (uint256) {
            uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

            // Validate the media type and associated data
            validateMediaType(metadata);

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/CultureIndex.sol#L209-L216

## Recommended Mitigation Steps

Add name, description and image validation in `createPiece` function.

# [L-4] Rereview voting clockMode when deploying to L2 such as Optimism

The clockmode in voting calculation is defaulted to block.number. However, there can be potential issue when the block.number is determined differently in Optimism.

        function CLOCK_MODE() public view virtual returns (string memory) {
            // Check that the clock was not modified
            if (clock() != Time.blockNumber()) {
                revert ERC6372InconsistentClock();
            }
            return "mode=blocknumber&from=default";
        }

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/base/VotesUpgradeable.sol#L97-L103

Similar issue is reported: https://github.com/code-423n4/2023-06-lybra-findings/issues/114

# [NC-1] Incorrect Natspec Documentation

The Natspec documentation for the `getPastVotes` function is misleading and incorrect. It currently describes the functionality of a different function (`getVotes`), which can cause confusion for developers and users of the code.

        /**
        * @notice Returns the voting power of a voter at the current block.
        * @param account The address of the voter.
        * @return The voting power of the voter.
        */
        function getPastVotes(address account, uint256 blockNumber) external view override returns (uint256) {
            return _getPastVotes(account, blockNumber);
        }

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/CultureIndex.sol#L269-L276

## Recommended Mitigation Steps

Update the Natspec comments to accurately reflect the purpose and behavior of the `getPastVotes` function.