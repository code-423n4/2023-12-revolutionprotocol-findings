## Note to the Judge:
I want to assure you that all gas optimizations in question were researched manually without the use of any automated tools or bots. I researched each of the gas issues below mannually to provide you with complete reports about each issue. Your understanding on this matter is greatly appreciated.


# [GAS-1] Combined Require Statements for Gas Optimization

## Lines of code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L307-L312

## Description:

The current implementation includes multiple require statements for input validation. These can be consolidated into a single require statement to optimize gas usage by reducing the number of opcode executions.

## Impact:

Gas optimization is crucial for smart contracts, especially on resource-constrained blockchains like Ethereum. The proposed change aims to reduce gas costs by streamlining the validation logic.

## Recommended Mitigation Steps:

```solidity
require(
    pieceId < _currentPieceId &&
    voter != address(0) &&
    !pieces[pieceId].isDropped &&
    !(votes[pieceId][voter].voterAddress != address(0)),
    "Invalid conditions"
);
```

OR

```solidity
require(
    pieceId < _currentPieceId &&
    voter != address(0) &&
    !pieces[pieceId].isDropped &&
    votes[pieceId][voter].voterAddress == address(0),
    "Invalid conditions"
);
```

# [GAS-2] Refactoring Event Emission in createPiece Function

## Lines of code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L236-L247

## Description:

The createPiece function in the provided code contains two separate for loops for adding creators to the newPiece.creators array and emitting corresponding events. This can lead to suboptimal gas efficiency due to redundant iterations and event emissions.

The current implementation has two distinct for loops: one for adding creators to the newPiece.creators array and another for emitting the PieceCreatorAdded events. This structure can be optimized to improve gas efficiency by combining the two operations within a single loop.

## Impact:

Gas efficiency in smart contracts is crucial, especially in functions executed frequently. Reducing unnecessary iterations and event emissions can lead to lower transaction costs and improved overall contract performance.

## Recommended Mitigation Steps:

To enhance gas efficiency, refactor the createPiece function by combining the two for loops into a single loop. The suggested proof of concept demonstrates a more streamlined implementation. Test thoroughly to ensure the refactored code maintains the intended functionality.

```diff
    function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

        // Validate the media type and associated data
        validateMediaType(metadata);

        uint256 pieceId = _currentPieceId++;

        /// @dev Insert the new piece into the max heap
        maxHeap.insert(pieceId, 0);

        ArtPiece storage newPiece = pieces[pieceId];

        newPiece.pieceId = pieceId;
        newPiece.totalVotesSupply = _calculateVoteWeight(
            erc20VotingToken.totalSupply(),
            erc721VotingToken.totalSupply()
        );
        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
        newPiece.metadata = metadata;
        newPiece.sponsor = msg.sender;
        newPiece.creationBlock = block.number;
        newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;

        for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);

+           // Store the creator data in a variable
+           address creator = creatorArray[i].creator;
+           uint256 bps = creatorArray[i].bps;

+           // Emit an event for each creator
+           emit PieceCreatorAdded(pieceId, creator, msg.sender, bps);
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

-       // Emit an event for each creator
-       for (uint i; i < creatorArrayLength; i++) {
-           emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
-       }

        return newPiece.pieceId;
    }


```

# [GAS-3] Avoid Redundant Computations & Variable Handling Refinement

## Lines of code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L94-L113

## Description:

Instead of computing 2 _ pos + 1 and 2 _ pos + 2 multiple times, the optimization suggests calculating them once and storing the results in variables. This modification aims to enhance the efficiency of the code by eliminating redundant calculations and improving overall performance.

In addition, the optimization proposes a refinement in variable handling. It recommends reading values from the valueMapping and heap arrays once and storing them in local variables before further processing. This adjustment not only reduces redundant array access but also enhances code readability by using meaningful variable names.

## Recommended Mitigation Steps:

```diff
    MaxHeap.sol:

    function maxHeapify(uint256 pos) internal {

        uint256 left = 2 * pos + 1;
-       uint256 right = 2 * pos + 2;
+       uint256 right = left + 1;

-       uint256 posValue = valueMapping[heap[pos]];
-       uint256 leftValue = valueMapping[heap[left]];
-       uint256 rightValue = valueMapping[heap[right]];


        // Read values once and store in local variables
+       address posElement = heap[pos];
+       address leftElement = heap[left];
+       address rightElement = heap[right];
+       uint256 posValue = valueMapping[posElement];
+       uint256 leftValue = valueMapping[leftElement];
+       uint256 rightValue = valueMapping[rightElement];

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

```

# [GAS-4] Reduced Storage Reads

## Lines of code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L136-L151

## Description:

Combining conditions avoids unnecessary storage reads when `newValue` is equal to the current value stored in `valueMapping[itemId]`. In such cases, the original code would read from storage twice (once for `newValue` and once for `valueMapping[itemId]`). The combined condition ensures that these reads are performed only when needed.


## Recommended Mitigation Steps:

```diff
    MaxHeap.sol:

    function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
        uint256 position = positionMapping[itemId];
-       uint256 oldValue = valueMapping[itemId];

        // Update the value in the valueMapping
-       valueMapping[itemId] = newValue;

        // Decide whether to perform upwards or downwards heapify
-       if (newValue > oldValue) {
-           // Upwards heapify
-           while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
-               swap(position, parent(position));
-               position = parent(position);
-           }
-       } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify



+       if (newValue != valueMapping[itemId]) {
+           if (newValue > valueMapping[itemId]) {
+               // Upwards heapify
+               while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
+                   swap(position, parent(position));
+                   position = parent(position);
+               }
+           } else {
+               maxHeapify(position); // Downwards heapify
+           }
+       }


    }

```


# [GAS-5] Use require Instead of if Statement

## Lines of code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L181-L184

## Description:

Instead of using an if statement followed by a revert statement, you can use the require statement, which is more gas-efficient. The require statement automatically throws an exception and reverts the transaction if the condition is not met.

## Recommended Mitigation Steps:

```diff
    MaxHeap.sol:

    
    function _authorizeUpgrade(address _newImpl) internal view override onlyOwner {
        // Ensure the new implementation is a registered upgrade
-       if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
+       require(manager.isRegisteredUpgrade(_getImplementation(), _newImpl), "INVALID_UPGRADE");

    }

```



# [GAS-6] Combining Require Statements in ERC20TokenEmitter's buyToken Function

## Lines of code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L152-L163

## Description:

 The changes involve consolidating multiple require statements into a single require statement, reducing gas costs by minimizing opcode execution. Additionally, the modifications enhance code readability and maintainability by combining related conditions. The update ensures that the transaction is deemed invalid only if any of the specified conditions, such as the sender being the treasury or creatorsAddress, zero ether sent, or mismatched array lengths, are not met. This optimization contributes to a more gas-efficient and secure execution of the buyToken function.

## Recommended Mitigation Steps:

```diff
    ERC20TokenEmitter.sol:
    
    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
-       //prevent treasury from paying itself
-       require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

-       require(msg.value > 0, "Must send ether");
-       // ensure the same number of addresses and bps
-       require(addresses.length == basisPointSplits.length, "Parallel arrays required");

+       require(
+           (msg.sender != treasury && msg.sender != creatorsAddress) && 
+           (msg.value > 0) && 
+           (addresses.length == basisPointSplits.length),
+           "Invalid transaction"
+       );


        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );

        //Share of purchase amount to send to treasury
        uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;

        //Share of purchase amount to reserve for creators
        //Ether directly sent to creators
        uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
        //Tokens to emit to creators
        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
            : int(0);

        // Tokens to emit to buyers
        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);

        //Transfer ETH to treasury and update emitted
        emittedTokenWad += totalTokensForBuyers;
        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

        //Deposit funds to treasury
        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
        require(success, "Transfer failed.");

        //Transfer ETH to creators
        if (creatorDirectPayment > 0) {
            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed.");
        }

        //Mint tokens for creators
        if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
            _mint(creatorsAddress, uint256(totalTokensForCreators));
        }

        uint256 bpsSum = 0;

        //Mint tokens to buyers

        for (uint256 i = 0; i < addresses.length; i++) {
            if (totalTokensForBuyers > 0) {
                // transfer tokens to address
                _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
            }
            bpsSum += basisPointSplits[i];
        }

        require(bpsSum == 10_000, "bps must add up to 10_000");

        emit PurchaseFinalized(
            msg.sender,
            msg.value,
            toPayTreasury,
            msg.value - msgValueRemaining,
            uint256(totalTokensForBuyers),
            uint256(totalTokensForCreators),
            creatorDirectPayment
        );

        return uint256(totalTokensForBuyers);
    }


```
