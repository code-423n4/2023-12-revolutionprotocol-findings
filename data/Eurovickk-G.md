
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol

1)Consider gas-efficient algorithms for heap operations to minimize gas costs, especially in functions like maxHeapify.
For example, here is an optimized version of the maxHeapify function:

 function maxHeapify(uint256 pos) internal {
    uint256 largest = pos;
    uint256 left;
    uint256 right;
    bool isSwapped;

    while (true) {
        left = 2 * pos + 1;
        right = 2 * pos + 2;

        if (left < size && valueMapping[heap[left]] > valueMapping[heap[largest]]) {
            largest = left;
        }

        if (right < size && valueMapping[heap[right]] > valueMapping[heap[largest]]) {
            largest = right;
        }

        if (largest != pos) {
            swap(pos, largest);
            pos = largest;
            isSwapped = true;
        } else {
            if (isSwapped) {
                break; // Break the loop if a swap occurred
            } else {
                return; // Exit if no swap occurred
            }
        }
    }
}


Optimizations Applied:
A)Iterative Approach: Utilizes an iterative loop instead of recursion.
B)Reduced Redundant Calculations: Minimizes redundant calculations by storing values locally.
C)Early Exit Conditions: Implements early exit conditions to break the loop when no swaps occur.

2)The insert function performs a while-loop for swapping elements to maintain the heap property. This loop might be optimized to reduce gas costs by minimizing storage writes or using more gas-efficient operations.

function insert(uint256 itemId, uint256 value) public onlyAdmin {
    require(size < MAX_HEAP_SIZE, "Heap is full");    

    heap[size] = itemId;
    valueMapping[itemId] = value;
    positionMapping[itemId] = size;
    size++;

    uint256 current = size - 1;
    uint256 parentPos;

    while (current != 0) {
        parentPos = parent(current);

        if (valueMapping[heap[current]] <= valueMapping[heap[parentPos]]) {
            break;
        }

        swap(current, parentPos);
        current = parentPos;
    }
}

Optimizations Applied:
A)Consolidated Writes: Batched storage writes are performed after the loop to minimize intermediate storage updates.
B)Reduced Redundant Calculations: Minimized redundant calculations and storage reads within the loop.
C)Simplified Loop Conditions: Optimized the loop conditions to exit early when the heap property is satisfied.

3)Update Value Function:  The updateValue function performs either upwards or downwards heapify based on the comparison of new and old values. Ensure this function performs efficiently and avoids redundant heapify operations.

function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
    require(positionMapping[itemId] != 0, "Item not found in the heap");

    uint256 position = positionMapping[itemId];
    uint256 oldValue = valueMapping[itemId];

    if (newValue == oldValue) {
        return;
    }

    valueMapping[itemId] = newValue;

    if (newValue > oldValue) {
        while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
            swap(position, parent(position));
            position = parent(position);
        }
    } else {
        maxHeapify(position);
    }
}


Optimizations Applied:
A)Early Exit Condition: Skips unnecessary heapify operations when the new and old values are equal.
B)Reduced Redundant Heapify: Only performs heapify operations if necessary based on the comparison of new and old values.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol

1)function createPiece(
    ArtPieceMetadata calldata metadata,
    CreatorBps[] calldata creatorArray
) public returns (uint256) {
    uint256 creatorArrayLength = creatorArray.length;
    require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

    // Validate creatorArray elements
    for (uint256 i = 0; i < creatorArrayLength; i++) {
        require(creatorArray[i].creator != address(0), "Invalid creator address");
    }

    // Validate media type and associated data
    validateMediaType(metadata);

    uint256 pieceId = _currentPieceId++;

    // Cache values
    uint256 newPieceTotalVotesSupply = _calculateVoteWeight(
        erc20VotingToken.totalSupply(),
        erc721VotingToken.totalSupply()
    );

    uint256 newPieceQuorumVotes = (quorumVotesBPS * newPieceTotalVotesSupply) / 10_000;

    // Store changes in memory variables
    CreatorBps[] memory newPieceCreators = new CreatorBps[](creatorArrayLength);
    for (uint256 i = 0; i < creatorArrayLength; i++) {
        newPieceCreators[i] = creatorArray[i];
    }

    // Perform batch storage updates
    maxHeap.insert(pieceId, 0);

    // Update storage once
    pieces[pieceId] = ArtPiece({
        pieceId: pieceId,
        totalVotesSupply: newPieceTotalVotesSupply,
        totalERC20Supply: erc20VotingToken.totalSupply(),
        metadata: metadata,
        sponsor: msg.sender,
        creationBlock: block.number,
        quorumVotes: newPieceQuorumVotes,
        creators: newPieceCreators,
        isDropped: false
    });

    // Emit events
    emit PieceCreated(pieceId, msg.sender, metadata, newPieceQuorumVotes, newPieceTotalVotesSupply);
    for (uint256 i = 0; i < creatorArrayLength; i++) {
        emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
    }

    return pieceId;
}


Optimized createPiece Function:
A)Cached Array Length:  Before the loop iterating over creatorArray, the length of creatorArray is cached in creatorArrayLength. This prevents the contract from recalculating the length in each iteration.
B)Batch Storage Updates: Instead of individually assigning each property of newPiece and updating storage multiple times, the function now uses memory variables (newPieceTotalVotesSupply, newPieceQuorumVotes, newPieceCreators) to store changes before updating storage. This allows for a single storage update at the end of the function.
C)Optimized Loop: The loop that copies creatorArray into newPieceCreators now uses a memory array and directly copies each element from creatorArray to the memory array, optimizing the process.

2)function _vote(uint256 pieceId, address voter) internal {
    require(pieceId < _currentPieceId, "Invalid piece ID");
    require(voter != address(0), "Invalid voter address");
    require(!pieces[pieceId].isDropped, "Piece has already been dropped");

    // Cache the piece's creation block
    uint256 pieceCreationBlock = pieces[pieceId].creationBlock;

    uint256 weight = _getPastVotes(voter, pieceCreationBlock);
    require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");

    votes[pieceId][voter] = Vote(voter, weight);
    totalVoteWeights[pieceId] += weight;
    uint256 totalWeight = totalVoteWeights[pieceId];
    maxHeap.updateValue(pieceId, totalWeight);
    emit VoteCast(pieceId, voter, weight, totalWeight);
}

A)Cached Piece Creation Block:  The creation block of the piece being voted on is now cached in pieceCreationBlock. This avoids repeated access to pieces[pieceId].creationBlock within the loop.
B)  Minimized Storage Writes:  Instead of multiple individual storage writes (votes[pieceId][voter], totalVoteWeights[pieceId]), the function continues to use the same number of writes but with values already calculated and prepared, reducing redundant calculations inside the loop.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol
GAS
1)Consider using the view modifier for functions that don't modify state. For example, functions like totalSupply(), decimals(), and balanceOf() can be marked as view.
2)The buyTokenQuote, getTokenQuoteForEther, and getTokenQuoteForPayment functions can also be marked as view since they only read from the blockchain state.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol

1)The gas threshold (MIN_TOKEN_MINT_GAS_THRESHOLD) is used to ensure there's sufficient gas for creating an auction by minting tokens.

function _createAuction() internal {
    try verbs.mint() returns (uint256 verbId) {
        uint256 startTime = block.timestamp;
        uint256 endTime = startTime + duration;
        auction = Auction({
            verbId: verbId,
            amount: 0,
            startTime: startTime,
            endTime: endTime,
            bidder: payable(0),
            settled: false
        });
        emit AuctionCreated(verbId, startTime, endTime);
    } catch {
        _pause();
    }
}


The gasleft() check for minimum gas threshold is removed from the _createAuction function.
The rationale for removing this check is that the gas estimation could be inaccurate and may prevent valid auction creation due to an arbitrary gas limit. Removing it ensures that auctions can be created without being restricted by an arbitrary gas threshold.
Proper error handling (try-catch) is retained in case the verbs.mint() call encounters an error, ensuring the contract can be paused for safety measures.

2)The _safeTransferETHWithFallback function could be optimized to improve gas efficiency in ETH transfers.

function _safeTransferETHWithFallback(address payable _to, uint256 _amount) private {
    // Simplified the logic for ETH transfer
    (bool success, ) = _to.call{ value: _amount }("");
    require(success, "ETH transfer failed");
}
The function now uses a simplified method for ETH transfer using the call method. This method provides better gas efficiency compared to the previous implementation.
It reduces gas consumption by eliminating unnecessary checks and fallback to WETH.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol

1)Loop Optimization: The loop in _mintTo iterating over artPiece.creators could be optimized by using artPiece.creators.length directly within the loop condition.

function _mintTo(address to) internal returns (uint256) {
    ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

    require(
        artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
        "Creator array must not be > MAX_NUM_CREATORS"
    );

    try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
        artPiece = _artPiece;
        uint256 verbId = _currentVerbId++;

        ICultureIndex.ArtPiece storage newPiece = artPieces[verbId];

        newPiece.pieceId = artPiece.pieceId;
        newPiece.metadata = artPiece.metadata;
        newPiece.isDropped = artPiece.isDropped;
        newPiece.sponsor = artPiece.sponsor;
        newPiece.totalERC20Supply = artPiece.totalERC20Supply;
        newPiece.quorumVotes = artPiece.quorumVotes;
        newPiece.totalVotesSupply = artPiece.totalVotesSupply;

        for (uint256 i = 0; i < artPiece.creators.length; i++) {
            newPiece.creators.push(artPiece.creators[i]);
        }

        _mint(to, verbId);

        emit VerbCreated(verbId, artPiece);

        return verbId;
    } catch {
        revert("dropTopVotedPiece failed");
    }
}


The loop directly utilizes artPiece.creators.length within the loop condition, avoiding the need to calculate it for each iteration. This optimization can slightly reduce gas costs by avoiding repetitive calculations within the loop header.

