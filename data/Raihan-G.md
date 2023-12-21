# GAS OPTIMIZATION

### Note: These are instances of ([G‑01],[G‑02],[G‑03],[G‑04]) the automated report missed

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|State variables should be cached in stack variables rather than re-reading them from storage|5|
|[G‑02]|Multiple accesses of a mapping/array should use a local variable cache|6|
|[G‑03]|Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate|1|
|[G‑04]|Avoid transferring amounts of zero in order to save gas|1|
|[G‑05]|Optimize External Calls with Assembly for Memory Efficiency|2|
|[G‑06]|Sort Solidity operations using short-circuit mode|1|
|[G‑07]|Do not shrink Variables|1|
|[G‑08]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|1|
|[G‑09]|Most important: avoid zero to one storage writes where possible|1|
|[G‑10]|Use assembly to perform efficient back-to-back calls|2|
|[G‑11]|`keccak256()` should only need to be called on a specific string literal once|1|
|[G‑12]|Use selfbalance() instead of address(this).balance|2|
|[G‑13]|Cache external calls outside of loop to avoid re-calling function on each iteration|1|
|[G‑14]|"" has the same value as new bytes(0) but costs less gas|2|
|[G‑15]|Unlimited gas consumption risk due to external call recipients|2|
|[G‑16]|When variables are declared with the storage keyword ,caching any fields that need to be re-read in stack variables Saves gas|2|
|[G‑17]|Keep strings smaller than 32 bytes|1|
|[G‑18]|Use fallback or receive instead of deposit() when transferring Ether|1|
|[G‑19]|Using assembly to revert with an error message|78|
|[G‑20]|Split revert statements|2|
|[G‑21]|It is sometimes cheaper to cache calldata|1|
|[G‑22]|Don’t cache calls that are only used once to save gas|1|
|[G‑23]|Change public state variable visibility to private|3|
|[G‑24]|Use uint256(1)/uint256(2) instead for true and false boolean states|3|
|[G‑25]|Understand the trade-offs when choosing between internal functions and modifiers|4|
|[G‑26]|Using private for immutable to saves gas|6|
|[G‑27]|Move storage pointer to top of function to avoid offset calculation|1|
|[G‑28]|Use assembly for math (add, sub, mul, div)|1|
|[G‑29]|Save gas by preventing zero amount in mint() and burn()|2|
|[G‑30]|Do not perform depositRewards when totalReward is 0 to save gas|1|
|[G‑31]|Do not perform deposit when _amount is 0 to save gas|1|
|[G‑32]|Refactor a modifier to call a local function instead of directly having the code in the modifier, saving bytecode size and thereby deployment cost|5|


 


## [G-01] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### Note: These are instances the automated report missed

---
### [G-1-1] Optimization: Caching timeBuffer State Variable for Gas Efficiency in createBid Function
#### Note: These are instances the automated report missed
```solidity
File: packages/revolution/src/AuctionHouse.sol
191      bool extended = _auction.endTime - block.timestamp < timeBuffer;
192      if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L191-L192

- #### In this updated version, the timeBuffer state variable is cached in the _timeBuffer local variable. By doing so, we avoid reading the state variable multiple times, resulting in gas savings.
```diff
    function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
        IAuctionHouse.Auction memory _auction = auction;
      
        //require bidder is valid address
        require(bidder != address(0), "Bidder cannot be zero address");
        require(_auction.verbId == verbId, "Verb not up for auction");
        //slither-disable-next-line timestamp
        require(block.timestamp < _auction.endTime, "Auction expired");
        require(msg.value >= reservePrice, "Must send at least reservePrice");
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );

        address payable lastBidder = _auction.bidder;

        auction.amount = msg.value;
        auction.bidder = payable(bidder);
+       uint256 _timeBuffer = timeBuffer;
        // Extend the auction if the bid was received within `timeBuffer` of the auction end time
-       bool extended = _auction.endTime - block.timestamp < timeBuffer;
+       bool extended = _auction.endTime - block.timestamp < _timeBuffer;
-       if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
+       if (extended) auction.endTime = _auction.endTime = block.timestamp + _timeBuffer;

        // Refund the last bidder, if applicable
        if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);

        emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);

        if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);
    }
```

---
### [G-1-2] Optimization: Caching entropyRateBps State Variable in _settleAuction Function
#### Note: These are instances the automated report missed
```solidity
File: packages/revolution/src/AuctionHouse.sol
383  if (creatorsShare > 0 && entropyRateBps > 0) {

390  uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);    
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L383

- #### In this updated version, the entropyRateBps state variable is cached in the entropyRate local variable. By doing so, we avoid reading the state variable multiple times, resulting in gas savings.

```diff
    function _settleAuction() internal {
        IAuctionHouse.Auction memory _auction = auction;

        require(_auction.startTime != 0, "Auction hasn't begun");
        require(!_auction.settled, "Auction has already been settled");
        //slither-disable-next-line timestamp
        require(block.timestamp >= _auction.endTime, "Auction hasn't completed");

        auction.settled = true;

        uint256 creatorTokensEmitted = 0;
        // Check if contract balance is greater than reserve price
        if (address(this).balance < reservePrice) {
            // If contract balance is less than reserve price, refund to the last bidder
            if (_auction.bidder != address(0)) {
                _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
            }

            // And then burn the Noun
            verbs.burn(_auction.verbId);
        } else {
            //If no one has bid, burn the Verb
            if (_auction.bidder == address(0))
                verbs.burn(_auction.verbId);
                //If someone has bid, transfer the Verb to the winning bidder
            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

            if (_auction.amount > 0) {
                // Ether going to owner of the auction
                uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;

                //Total amount of ether going to creator
                uint256 creatorsShare = _auction.amount - auctioneerPayment;

                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

                //Build arrays for erc20TokenEmitter.buyToken
                uint256[] memory vrgdaSplits = new uint256[](numCreators);
                address[] memory vrgdaReceivers = new address[](numCreators);

                //Transfer auction amount to the DAO treasury
                _safeTransferETHWithFallback(owner(), auctioneerPayment);

                uint256 ethPaidToCreators = 0;

+               // Cache the entropyRateBps state variable
+               uint256 entropyRate = entropyRateBps;

                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
-               if (creatorsShare > 0 && entropyRateBps > 0) {
+               if (creatorsShare > 0 && entropyRate > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
                        vrgdaReceivers[i] = creator.creator;
                        vrgdaSplits[i] = creator.bps;

                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
-                       uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
+                       uint256 paymentAmount = (creatorsShare * entropyRate * creator.bps) / (10_000 * 10_000);
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
                    }
                }

                //Buy token from ERC20TokenEmitter for all the creators
                if (creatorsShare > ethPaidToCreators) {
                    creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: creatorsShare - ethPaidToCreators }(
                        vrgdaReceivers,
                        vrgdaSplits,
                        IERC20TokenEmitter.ProtocolRewardAddresses({
                            builder: address(0),
                            purchaseReferral: address(0),
                            deployer: deployer
                        })
                    );
                }
            }
        }

        emit AuctionSettled(_auction.verbId, _auction.bidder, _auction.amount, creatorTokensEmitted);
    }
```



---


### [G-1-3] Optimization: Caching creatorsAddress State Variable in buyToken Function
#### Note: These are instances the automated report missed
```solidity
File: packages/revolution/src/ERC20TokenEmitter.sol
158  require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

196  (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));

201     if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
202         _mint(creatorsAddress, uint256(totalTokensForCreators));
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L158

- #### In this updated version, the creatorsAddress state variable is cached in the creatorsAddr local variable. By doing so, we avoid reading the state variable multiple times, resulting in gas savings.
```diff
    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
+       // Cache the creatorsAddress state variable
+       address creatorsAddr = creatorsAddress;
        //prevent treasury from paying itself
-       require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");
+       require(msg.sender != treasury && msg.sender != creatorsAddr, "Funds recipient cannot buy tokens");

        require(msg.value > 0, "Must send ether");
        // ensure the same number of addresses and bps
        require(addresses.length == basisPointSplits.length, "Parallel arrays required");

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
-           (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
+           (success, ) = creatorsAddr.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed.");
        }

        //Mint tokens for creators
-       if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
+       if (totalTokensForCreators > 0 && creatorsAddr != address(0)) {
-           _mint(creatorsAddress, uint256(totalTokensForCreators));
+           _mint(creatorsAddr, uint256(totalTokensForCreators));
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
---
### [G-1-4] Optimization: Caching size State Variable in maxHeapify Function
#### Note: These are instances the automated report missed
```solidity
File: packages/revolution/src/MaxHeap.sol
102   if (pos >= (size / 2) && pos <= size) return;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L102


- #### In this updated version, the size state variable is cached in the heapSize local variable. By doing so, we avoid reading the state variable multiple times, resulting in gas savings.
```diff
    function maxHeapify(uint256 pos) internal {
+       uint256 heapSize = size; // Cache the size state variable
        uint256 left = 2 * pos + 1;
        uint256 right = 2 * pos + 2;

        uint256 posValue = valueMapping[heap[pos]];
        uint256 leftValue = valueMapping[heap[left]];
        uint256 rightValue = valueMapping[heap[right]];

-       if (pos >= (size / 2) && pos <= size) return;
+       if (pos >= (heapSize / 2) && pos <= heapSize) return;

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

---
### [G-1-5] Optimization: Caching size State Variable in insert Function
#### Note: These are instances the automated report missed

```solidity
File: packages/revolution/src/MaxHeap.sol
        heap[size] = itemId;
        valueMapping[itemId] = value; // Update the value mapping
        positionMapping[itemId] = size; // Update the position mapping

        uint256 current = size;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L120-L124

- #### By caching the size state variable in the heapSize local variable, we avoid repeated reads to the state variable, resulting in improved gas efficiency.
```diff
    function insert(uint256 itemId, uint256 value) public onlyAdmin {
+       uint256 heapSize = size; // Cache the size state variable
-       heap[size] = itemId;
+       heap[heapSize] = itemId;
        valueMapping[itemId] = value; // Update the value mapping
-       positionMapping[itemId] = size; // Update the position mapping
+       positionMapping[itemId] = heapSize; // Update the position mapping

-       uint256 current = size;
+       uint256 current = heapSize;
        while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
            swap(current, parent(current));
            current = parent(current);
        }
-       size++;
+       size = heapSize + 1; // Update the size state variable
    }
```

---



## [G-02] Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

#### Note: These are instances the automated report missed


- #### Caching valueMapping and heap mappings locally in the maxHeapify function optimizes gas usage by reducing redundant storage lookups.
```solidity
File: packages/revolution/src/MaxHeap.sol
98       uint256 posValue = valueMapping[heap[pos]];
99       uint256 leftValue = valueMapping[heap[left]];
100      uint256 rightValue = valueMapping[heap[right]];
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L98-L100




- #### To optimize gas usage in the updateValue function, caching the valueMapping mapping locally can be employed to reduce redundant storage lookups.

```solidity
File: packages/revolution/src/MaxHeap.sol
        uint256 oldValue = valueMapping[itemId];

        // Update the value in the valueMapping
        valueMapping[itemId] = newValue;

        // Decide whether to perform upwards or downwards heapify
        if (newValue > oldValue) {
            // Upwards heapify
            while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L138-L146

## [G-03] Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

#### Note: These are instances the automated report missed

```solidity
File: packages/revolution/src/MaxHeap.sol
65  mapping(uint256 => uint256) public heap;

67  uint256 public size = 0;

    /// @notice Mapping to keep track of the value of an item in the heap
70  mapping(uint256 => uint256) public valueMapping;

    /// @notice Mapping to keep track of the position of an item in the heap
73  mapping(uint256 => uint256) public positionMapping;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L65-L73

### Combining Multiple Mappings into a Single Mapping in Solidity Contract like this
```solidity
    struct MappingData {
        uint256 heapValue;
        uint256 valueMappingValue;
        uint256 positionMappingValue;
    }

    mapping(uint256 => MappingData) public combinedMapping;
```



## [G-04] Avoid transferring amounts of zero in order to save gas
Skipping the external call when nothing will be transferred, will save at least 100 gas

#### Note: These are instances the automated report missed

```solidity
File: packages/revolution/src/AuctionHouse.sol
438  bool wethSuccess = IWETH(WETH).transfer(_to, _amount);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L438




## [G-05] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.
Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.
Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.[Reffrence](https://github.com/code-423n4/2023-10-ethena/blob/main/bot-report.md#g-10-optimize-external-calls-with-assembly-for-memory-efficiency
)

- ### these are interfaces external calls in `AuctionHouse._settleAuction` function
```solidity
File: packages/revolution/src/AuctionHouse.sol
355  verbs.burn(_auction.verbId);

359  verbs.burn(_auction.verbId);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L355


## [G-06] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

In the MaxHeap.maxHeapify function, the first if condition `pos >= (size / 2)` consumes more gas compared to the condition `pos <= size`.
```solidity
File: packages/revolution/src/MaxHeap.sol
102   if (pos >= (size / 2) && pos <= size) return;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L102

- #### Reordering the conditions in `MaxHeap.maxHeapify` to prioritize `pos <= size` over `pos >= (size / 2)` allows for efficient short-circuiting. If the low gas cost operation is true, the evaluation of the high gas cost operation can be bypassed. This optimization helps reduce gas consumption during execution.
```diff
- if (pos >= (size / 2) && pos <= size) return;
+ if (pos <= size && pos >= (size / 2)) return;
```

## [G-07] Do not shrink Variables
This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.


```solidity
File: packages/revolution/src/AuctionHouse.sol
63  uint8 public minBidIncrementPercentage;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L63

While uint8 variables occupy less storage space, the Ethereum Virtual Machine (EVM) requires additional gas to convert them to uint256 for performing operations. This conversion process incurs extra gas costs, which can be avoided by using uint256 directly


## [G-08] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant
The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.


There are 1 instances of this issue:


```solidity
File: packages/revolution/src/CultureIndex.sol
29 bytes32 public constant VOTE_TYPEHASH =
30        keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L29-L30


## [G-09] Most important: avoid zero to one storage writes where possible
Initializing a storage variable is one of the most expensive operations a contract can do.

When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.

- `size` is storage variable
```solidity
File: packages/revolution/src/MaxHeap.sol
67  uint256 public size = 0;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L67




## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

check this report for implementation
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)

- #### verbs.getArtPieceById is external call
```solidity
File: packages/revolution/src/AuctionHouse.sol
370          uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
371          address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L370-L371

## [G-11] `keccak256()` should only need to be called on a specific string literal once
It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once


```solidity
File: packages/revolution/src/CultureIndex.sol
30        keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L29-L30

## [G-12] Use selfbalance() instead of address(this).balance
it's recommended to use the selfbalance() function instead of address(this).balance. The selfbalance() function is a built-in Solidity function that returns the balance of the current contract in Wei and is considered more gas-efficient and secure.

```solidity
File: packages/revolution/src/AuctionHouse.sol
348  if (address(this).balance < reservePrice) {

421  if (address(this).balance < _amount) revert("Insufficient balance");          
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L348

## [G-13] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

- #### `verbs.getArtPieceById(_auction.verbId)` is external call cache this outside of loop then use to save gas
```solidity
File: packages/revolution/src/AuctionHouse.sol
284    for (uint256 i = 0; i < numCreators; i++) {
385           ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];      
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L384-L385

## [G‑14] "" has the same value as new bytes(0) but costs less gas
In Solidity, new bytes(0) and "" both represent an empty byte array. However, "" is more gas-efficient than new bytes(0) This is because "" is a literal and is stored in the code, whereas new bytes(0) is a dynamic allocation that requires the creation of a new memory location at runtime

```solidity
File: packages/revolution/src/ERC20TokenEmitter.sol
191  (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));

196  (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L191

```diff
- 191  (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
+ 191  (bool success, ) = treasury.call{ value: toPayTreasury }("");


- 196  (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
+ 196  (success, ) = creatorsAddress.call{ value: creatorDirectPayment }("");
```


## [G‑15] Unlimited gas consumption risk due to external call recipients
When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted. To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.

```solidity
File: packages/revolution/src/ERC20TokenEmitter.sol
191  (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));

196  (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L191

- #### In the modified code, a gas limit of 50000 (you can adjust this value as per your requirements) is explicitly set for the external calls to treasury and creatorsAddress. This helps prevent the called contracts from consuming all the remaining gas and causing the transaction to revert.

```diff
- 191  (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
+ 191  (bool success, ) = treasury.call{ value: toPayTreasury, gas: 50000 }(new bytes(0));


- 196  (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
+ 196  (success, ) = creatorsAddress.call{ value: creatorDirectPayment, gas: 50000 }(new bytes(0));
```


## [G-16] When variables are declared with the storage keyword ,caching any fields that need to be re-read in stack variables Saves gas
[Reffrence](https://github.com/code-423n4/2023-07-arcade-findings/blob/main/data/c3phas-G.md#g-02when--variables-are-declared--with-the-storage-keyword-caching-any-fields-that-need-to-be-re-read-in-stack-variables-saves-gas)


- #### We can also cache `newPiece.totalVotesSupply` instead of accessing it more than once
```solidity
File: packages/revolution/src/CultureIndex.sol
234  newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;

240  emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L234-L240


## [G-17] Keep strings smaller than 32 bytes
In Solidity, strings are variable length dynamic data types, meaning their length can change and grow as needed.

If the length is 32 bytes or longer, the slot in which they are defined stores the length of the string * 2 + 1, while their actual data is stored elsewhere (the keccak hash of that slot).

However, if a string is less than 32 bytes, the length * 2 is stored at the least significant byte of it’s storage slot and the actual data of the string is stored starting from the most significant byte in the slot in which it is defined.


for more details check [this](https://www.rareskills.io/post/gas-optimization#viewer-dkmii:~:text=5.-,Keep%20strings%20smaller%20than%2032%20bytes,-In%20Solidity%2C%20strings) 

```solidity
File: packages/revolution/src/CultureIndex.sol
57  string public name;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L57

## [G-18] Use fallback or receive instead of deposit() when transferring Ether
Similar to above, you can “just transfer” ether to a contract and have it respond to the transfer instead of using a payable function. This of course, depends on the rest of the contract’s architecture.


Example Deposit in AAVE
```
contract AddLiquidity{

    receive() external payable {
      IWETH(weth).deposit{msg.value}();
      AAVE.deposit(weth, msg.value, msg.sender, REFERRAL_CODE)
    }
}
```
The fallback function is capable of receiving bytes data which can be parsed with abi.decode. This servers as an alternative to supplying arguments to a deposit function.

```solidity
File: packages/revolution/src/AuctionHouse.sol
435   IWETH(WETH).deposit{ value: _amount }();
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L355


## [G-19] Using assembly to revert with an error message
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.


Here’s an example;
```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```
From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.





There are 78 instances of this issue:

```solidity
File: packages/protocol-rewards/src/abstract/RewardSplits.sol

30  if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/protocol-rewards/src/abstract/RewardSplits.sol#L30-L30

```solidity

File: packages/revolution/src/AuctionHouse.sol

120  require(msg.sender == address(manager), "Only manager can initialize");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L120-L120

```solidity

File: packages/revolution/src/AuctionHouse.sol

121  require(_weth != address(0), "WETH cannot be zero address");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L121-L121

```solidity

File: packages/revolution/src/AuctionHouse.sol

129  require(
130      _auctionParams.creatorRateBps >= _auctionParams.minCreatorRateBps,
131      "Creator rate must be greater than or equal to the creator rate"
132  );


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L129-L132

```solidity

File: packages/revolution/src/AuctionHouse.sol

175  require(bidder != address(0), "Bidder cannot be zero address");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L175-L175

```solidity

File: packages/revolution/src/AuctionHouse.sol

176  require(_auction.verbId == verbId, "Verb not up for auction");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L176-L176

```solidity

File: packages/revolution/src/AuctionHouse.sol

178  require(block.timestamp < _auction.endTime, "Auction expired");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L178-L178

```solidity

File: packages/revolution/src/AuctionHouse.sol

179  require(msg.value >= reservePrice, "Must send at least reservePrice");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L179-L179

```solidity
File: packages/revolution/src/AuctionHouse.sol

180  require(
181      msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
182      "Must send more than last bid by minBidIncrementPercentage amount"
183  );
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L180-L183

```solidity
File: packages/revolution/src/AuctionHouse.sol
218  require(
219      _creatorRateBps >= minCreatorRateBps,
220      "Creator rate must be greater than or equal to minCreatorRateBps"
221  );


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L218-L221

```solidity
File: packages/revolution/src/AuctionHouse.sol
222  require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L222-L222

```solidity
File: packages/revolution/src/AuctionHouse.sol
234  require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L234-L234

```solidity
File: packages/revolution/src/AuctionHouse.sol
235  require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L235-L235

```solidity

File: packages/revolution/src/AuctionHouse.sol

238  require(
239      _minCreatorRateBps > minCreatorRateBps,
240      "Min creator rate must be greater than previous minCreatorRateBps"
241  );


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L238-L241

```solidity

File: packages/revolution/src/AuctionHouse.sol

254  require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L254-L254

```solidity

File: packages/revolution/src/AuctionHouse.sol

311  require(gasleft() >= MIN_TOKEN_MINT_GAS_THRESHOLD, "Insufficient gas for creating auction");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L311-L311

```solidity

File: packages/revolution/src/AuctionHouse.sol

339  require(_auction.startTime != 0, "Auction hasn't begun");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L339-L339

```solidity

File: packages/revolution/src/AuctionHouse.sol

340  require(!_auction.settled, "Auction has already been settled");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L340-L340

```solidity

File: packages/revolution/src/AuctionHouse.sol

342  require(block.timestamp >= _auction.endTime, "Auction hasn't completed");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L342-L342

```solidity

File: packages/revolution/src/AuctionHouse.sol

421  if (address(this).balance < _amount) revert("Insufficient balance");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L421-L421

```solidity

File: packages/revolution/src/AuctionHouse.sol

441      if (!wethSuccess) revert("WETH transfer failed");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/AuctionHouse.sol#L441-L441

```solidity

File: packages/revolution/src/CultureIndex.sol

117  require(msg.sender == address(manager), "Only manager can initialize");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L117-L117

```solidity

File: packages/revolution/src/CultureIndex.sol

119  require(_cultureIndexParams.quorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "invalid quorum bps");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L119-L119

```solidity

File: packages/revolution/src/CultureIndex.sol

120  require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L120-L120

```solidity

File: packages/revolution/src/CultureIndex.sol

121  require(_erc721VotingToken != address(0), "invalid erc721 voting token");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L121-L121

```solidity

File: packages/revolution/src/CultureIndex.sol

122  require(_erc20VotingToken != address(0), "invalid erc20 voting token");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L122-L122

```solidity

File: packages/revolution/src/CultureIndex.sol

160  require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L160-L160

```solidity

File: packages/revolution/src/CultureIndex.sol

163      require(bytes(metadata.image).length > 0, "Image URL must be provided");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L163-L163

```solidity

File: packages/revolution/src/CultureIndex.sol

165      require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L165-L165

```solidity

File: packages/revolution/src/CultureIndex.sol

167      require(bytes(metadata.text).length > 0, "Text must be provided");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L167-L167

```solidity

File: packages/revolution/src/CultureIndex.sol

182  require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L182-L182

```solidity

File: packages/revolution/src/CultureIndex.sol

186      require(creatorArray[i].creator != address(0), "Invalid creator address");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L186-L186

```solidity

File: packages/revolution/src/CultureIndex.sol

190  require(totalBps == 10_000, "Total BPS must sum up to 10,000");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L190-L190

```solidity

File: packages/revolution/src/CultureIndex.sol

308  require(pieceId < _currentPieceId, "Invalid piece ID");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L308-L308

```solidity

File: packages/revolution/src/CultureIndex.sol

309  require(voter != address(0), "Invalid voter address");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L309-L309

```solidity

File: packages/revolution/src/CultureIndex.sol

310  require(!pieces[pieceId].isDropped, "Piece has already been dropped");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L310-L310

```solidity

File: packages/revolution/src/CultureIndex.sol

311  require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L311-L311

```solidity

File: packages/revolution/src/CultureIndex.sol

314  require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L314-L314

```solidity

File: packages/revolution/src/CultureIndex.sol

398  require(
399      len == pieceIds.length && len == deadline.length && len == v.length && len == r.length && len == s.length,
400      "Array lengths must match"
401  );


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L398-L401

```solidity

File: packages/revolution/src/CultureIndex.sol

427  require(deadline >= block.timestamp, "Signature expired");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L427-L427

```solidity

File: packages/revolution/src/CultureIndex.sol

452  require(pieceId < _currentPieceId, "Invalid piece ID");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L452-L452

```solidity

File: packages/revolution/src/CultureIndex.sol

462  require(pieceId < _currentPieceId, "Invalid piece ID");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L462-L462

```solidity

File: packages/revolution/src/CultureIndex.sol

487  require(maxHeap.size() > 0, "Culture index is empty");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L487-L487

```solidity

File: packages/revolution/src/CultureIndex.sol

499  require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L499-L499

```solidity

File: packages/revolution/src/CultureIndex.sol

520  require(msg.sender == dropperAdmin, "Only dropper can drop pieces");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L520-L520

```solidity

File: packages/revolution/src/CultureIndex.sol

523  require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/CultureIndex.sol#L523-L523

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

91  require(msg.sender == address(manager), "Only manager can initialize");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L91-L91

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

96  require(_treasury != address(0), "Invalid treasury address");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L96-L96

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

158  require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L158-L158

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

160  require(msg.value > 0, "Must send ether");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L160-L160

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

162  require(addresses.length == basisPointSplits.length, "Parallel arrays required");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L162-L162

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

192  require(success, "Transfer failed.");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L192-L192

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

197      require(success, "Transfer failed.");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L197-L197

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

217  require(bpsSum == 10_000, "bps must add up to 10_000");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L217-L217

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

238  require(amount > 0, "Amount must be greater than 0");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L238-L238

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

255  require(etherAmount > 0, "Ether amount must be greater than 0");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L255-L255

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

272  require(paymentAmount > 0, "Payment amount must be greater than 0");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L272-L272

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

289  require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L289-L289

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

300  require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L300-L300

```solidity

File: packages/revolution/src/ERC20TokenEmitter.sol

310  require(_creatorsAddress != address(0), "Invalid address");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/ERC20TokenEmitter.sol#L310-L310

```solidity

File: packages/revolution/src/MaxHeap.sol

42  require(msg.sender == admin, "Sender is not the admin");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/MaxHeap.sol#L42-L42

```solidity

File: packages/revolution/src/MaxHeap.sol

56  require(msg.sender == address(manager), "Only manager can initialize");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/MaxHeap.sol#L56-L56

```solidity

File: packages/revolution/src/MaxHeap.sol

79  require(pos != 0, "Position should not be zero");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/MaxHeap.sol#L79-L79

```solidity

File: packages/revolution/src/MaxHeap.sol

157  require(size > 0, "Heap is empty");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/MaxHeap.sol#L157-L157

```solidity

File: packages/revolution/src/MaxHeap.sol

170  require(size > 0, "Heap is empty");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/MaxHeap.sol#L170-L170

```solidity

File: packages/revolution/src/NontransferableERC20Votes.sol

69  require(msg.sender == address(manager), "Only manager can initialize");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/revolution/src/NontransferableERC20Votes.sol#L69-L69

```solidity
File: packages/revolution/src/VerbsToken.sol
76  require(!isMinterLocked, "Minter is locked");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L76-L76

```solidity
File: packages/revolution/src/VerbsToken.sol
84         require(!isCultureIndexLocked, "CultureIndex is locked");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L84

```solidity
File: packages/revolution/src/VerbsToken.sol
92        require(!isDescriptorLocked, "Descriptor is locked");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L92-L92

```solidity
File: packages/revolution/src/VerbsToken.sol
100         require(msg.sender == minter, "Sender is not the minter");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L100-L100

```solidity
File: packages/revolution/src/VerbsToken.sol
137        require(msg.sender == address(manager), "Only manager can initialize");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L137-L137

```solidity
File: packages/revolution/src/VerbsToken.sol

139  require(_minter != address(0), "Minter cannot be zero address");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L139-L139

```solidity

File: packages/revolution/src/VerbsToken.sol

140  require(_initialOwner != address(0), "Initial owner cannot be zero address");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L140-L140

```solidity

File: packages/revolution/src/VerbsToken.sol

210  require(_minter != address(0), "Minter cannot be zero address");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L210-L210

```solidity

File: packages/revolution/src/VerbsToken.sol

274  require(verbId <= _currentVerbId, "Invalid piece ID");


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L274-L274

```solidity

File: packages/revolution/src/VerbsToken.sol

286  require(
287      artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
288      "Creator array must not be > MAX_NUM_CREATORS"
289  );


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L286-L289

```solidity
File: packages/revolution/src/VerbsToken.sol
317      revert("dropTopVotedPiece failed");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L317-L317

```solidity
330  require(manager.isRegisteredUpgrade(_getImplementation(), _newImpl), "Invalid upgrade");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L330-L330



## [G-20] Split revert statements
Similar to splitting require statements, you will usually save some gas by not having a boolean operator in the if statement.
```
contract CustomErrorBoolLessEfficient {
    error BadValue();

    function requireGood(uint256 x) external pure {
        if (x < 10 || x > 20) {
            revert BadValue();
        }
    }
}

contract CustomErrorBoolEfficient {
    error TooLow();
    error TooHigh();

    function requireGood(uint256 x) external pure {
        if (x < 10) {
            revert TooLow();
        }
        if (x > 20) {
            revert TooHigh();
        }
    }
}
```




```solidity
File: packages/revolution/src/CultureIndex.sol
441  if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L441


```solidity
41  if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main//packages/protocol-rewards/src/abstract/RewardSplits.sol#L41


## [G-21] It is sometimes cheaper to cache calldata
Although the calldataload instruction is a cheap opcode, the solidity compiler will sometimes output cheaper code if you cache calldataload. This will not always be the case, so you should test both possibilities.
```
contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            sum += arr[i];
            unchecked {
                ++i;
            }
        }
    }
}

```


```solidity
File: packages/revolution/src/ERC20TokenEmitter.sol
153  address[] calldata addresses,
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L153


- #### we declare a `cachedAddresses` variable of type address[] memory to cache the entire addresses array. The `cachedAddresses` array can be accessed and iterated over later in `buyToken` function using a loop or any other desired logic.
```diff
function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        //prevent treasury from paying itself
        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

        require(msg.value > 0, "Must send ether");
        
+       // Cache the addresses array
+       address[] memory cachedAddresses = addresses;

        // ensure the same number of addresses and bps
-       require(addresses.length == basisPointSplits.length, "Parallel arrays required");
+       require(cachedAddresses.length == basisPointSplits.length, "Parallel arrays required");

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

-       for (uint256 i = 0; i < addresses.length; i++) {
-       for (uint256 i = 0; i < cachedAddresses.length; i++) {    
            if (totalTokensForBuyers > 0) {
                // transfer tokens to address
-               _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
+               _mint(cachedAddresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
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

## [G-22] Don’t cache calls that are only used once to save gas

```solidity
File: packages/revolution/src/CultureIndex.sol
433  bytes32 digest = _hashTypedDataV4(voteHash);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L433

- In this modified code, the digest variable is eliminated, and the result of _hashTypedDataV4(voteHash) is directly passed as the first argument to the ecrecover function. This way, the result of _hashTypedDataV4(voteHash) is used without storing it in an intermediate variable.

```diff
    function _verifyVoteSignature(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal returns (bool success) {
        require(deadline >= block.timestamp, "Signature expired");

        bytes32 voteHash;

        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

-       bytes32 digest = _hashTypedDataV4(voteHash);

-       address recoveredAddress = ecrecover(digest, v, r, s);
+       address recoveredAddress = ecrecover(_hashTypedDataV4(voteHash), v, r, s);

        // Ensure to address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

        return true;
    }

```

## [G-23] Change public state variable visibility to private
it's generally a good practice to limit the visibility of state variables to the minimum necessary level. This means that you should use the private visibility modifier for state variables whenever possible, and avoid using the public modifier unless it is absolutely necessary.

1.Security: Public state variables can be read and modified by anyone on the blockchain, which can make your contract vulnerable to attacks. By using the private modifier, you can limit access to your state variables and reduce the risk of malicious actors exploiting them.

2.Encapsulation: Using private state variables can help to encapsulate the internal workings of your contract and make it easier to reason about and maintain. By only exposing the necessary interfaces to the outside world, you can reduce the complexity and potential for errors in your contract.

3.Gas costs: Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency.

If you do not require other contracts to read these variables, consider making them private or internal.

```solidity
File: packages/revolution/src/CultureIndex.sol
78  address public dropperAdmin;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L78


```solidity
File: packages/revolution/src/MaxHeap.sol
16  address public admin;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L16


```solidity
File: packages/revolution/src/VerbsToken.sol
42  address public minter;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L42



## [G-24] Use uint256(1)/uint256(2) instead for true and false boolean states
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

```solidity
File: packages/revolution/src/VerbsToken.sol
221  isMinterLocked = true;

243  isDescriptorLocked = true;

263  isCultureIndexLocked = true;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L221


## [G-25] Understand the trade-offs when choosing between internal functions and modifiers
Modifiers inject its implementation bytecode where it is used while internal functions jump to the location in the runtime code where the its implementation is. This brings certain trade-offs to both options.

- Using modifiers more than once means repetitiveness and increase in size of the runtime code but reduces gas cost because of the absence of jumping to the internal function execution offset and jumping back to continue. This means that if runtime gas cost matter most to you, then modifiers should be your choice but if deployment gas cost and/or reducing the size of the creation code is most important to you then using internal functions will be best.

- However, modifiers have the tradeoff that they can only be executed at the start or end of a functon. This means executing it at the middle of a function wouldn’t be directly possible, at least not without internal functions which kill the original purpose. This affects it’s flexibility. Internal functions however can be called at any point in a function.

Example showing difference in gas cost using modifiers and an internal function
```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.19;

/** deployment gas cost: 195435
    gas per call:
              restrictedAction1: 28367
              restrictedAction2: 28377
              restrictedAction3: 28411
 */
 contract Modifier {
    address owner;
    uint256 val;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function restrictedAction1() external onlyOwner {
        val = 1;
    }

    function restrictedAction2() external onlyOwner {
        val = 2;
    }

    function restrictedAction3() external onlyOwner {
        val = 3;
    }
}



/** deployment gas cost: 159309
    gas per call:
              restrictedAction1: 28391
              restrictedAction2: 28401
              restrictedAction3: 28435
 */
 contract InternalFunction {
    address owner;
    uint256 val;

    constructor() {
        owner = msg.sender;
    }

    function onlyOwner() internal view {
        require(msg.sender == owner);
    }

    function restrictedAction1() external {
        onlyOwner();
        val = 1;
    }

    function restrictedAction2() external {
        onlyOwner();
        val = 2;
    }

    function restrictedAction3() external {
        onlyOwner();
        val = 3;
    }
}
```
| Operation          | Deployment | restrictedAction1 | restrictedAction2 | restrictedAction3 |
| ------------------ | ---------- | ----------------- | ----------------- | ----------------- |
| Modifiers          | 195435     | 28367             | 28377             | 28411             |
| Internal Functions | 159309     | 28391             | 28401             | 28435             |


```solidity
File: packages/revolution/src/VerbsToken.sol
    modifier whenMinterNotLocked() {
        require(!isMinterLocked, "Minter is locked");
        _;
    }

    /**
     * @notice Require that the CultureIndex has not been locked.
     */
    modifier whenCultureIndexNotLocked() {
        require(!isCultureIndexLocked, "CultureIndex is locked");
        _;
    }

    /**
     * @notice Require that the descriptor has not been locked.
     */
    modifier whenDescriptorNotLocked() {
        require(!isDescriptorLocked, "Descriptor is locked");
        _;
    }

    /**
     * @notice Require that the sender is the minter.
     */
    modifier onlyMinter() {
        require(msg.sender == minter, "Sender is not the minter");
        _;
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L75-L102


## [G-26] Using private for immutable to saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

Using public for immutable variable gas value: 704034
Using private for immutable variable gas value: 685260

```solidity
    int256 public immutable targetPrice;

    int256 public immutable perTimeUnit;

    int256 public immutable decayConstant;

    int256 public immutable priceDecayPercent;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L16-L22


## [G-27] Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

```solidity
File: packages/revolution/src/CultureIndex.sol
223  ArtPiece storage newPiece = pieces[pieceId];
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L223


```diff
    function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
+       uint256 pieceId = _currentPieceId++;
+       ArtPiece storage newPiece = pieces[pieceId];
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

        // Validate the media type and associated data
        validateMediaType(metadata);

-       uint256 pieceId = _currentPieceId++;

        /// @dev Insert the new piece into the max heap
        maxHeap.insert(pieceId, 0);

-       ArtPiece storage newPiece = pieces[pieceId];

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
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

        // Emit an event for each creator
        for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }

        return newPiece.pieceId;
    }
```


## [G-28] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:
```
//addition in SolidityCache multiple accesses of a mapping/array
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263

Subtraction
```
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```
Gas: 300
```
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263

Multiplication
```
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325
```
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

Division
```
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325

```
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265



```solidity
File: packages/revolution/src/CultureIndex.sol
285     return erc20Balance + (erc721Balance * erc721VotingTokenWeight * 1e18);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L285


```diff
function _calculateVoteWeight(uint256 erc20Balance, uint256 erc721Balance) internal view returns (uint256) {
-   return erc20Balance + (erc721Balance * erc721VotingTokenWeight * 1e18);
+   uint256 voteWeight;
+   assembly {
+       // Calculate erc721Balance * erc721VotingTokenWeight in assembly
+       let result := mul(erc721Balance, erc721VotingTokenWeight)

+       // Multiply the result by 1e18 (10^18) in assembly
+       voteWeight := mul(result, 1e18)

+       // Add erc20Balance to voteWeight
+       voteWeight := add(voteWeight, erc20Balance)
+   }

+   return voteWeight;
}
```


## [G‑29] Save gas by preventing zero amount in mint() and burn()
fixes provided aim to prevent zero amounts from being passed to the `mint()` and `_mint()` functions in the `ERC20TokenEmitter.sol `and `NontransferableERC20Votes.sol` contracts, respectively. By adding a condition to check if the amount is greater than zero before executing the minting operation, gas savings can be achieved by avoiding unnecessary function calls with zero amounts.
```solidity
File: packages/revolution/src/ERC20TokenEmitter.sol
109  token.mint(_to, _amount);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L109

- The updated code ensures that only valid non-zero amounts are used for minting tokens, thereby improving efficiency and preventing unnecessary gas consumption.
fix code
```diff
    function _mint(address _to, uint256 _amount) private {
-       token.mint(_to, _amount);
+       if (amount > 0){
+         token.mint(_to, _amount);
+       }  
    }
```


```solidity
File: packages/revolution/src/NontransferableERC20Votes.sol
135  _mint(account, amount);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L135

- The updated code ensures that only valid non-zero amounts are used for minting tokens, thereby improving efficiency and preventing unnecessary gas consumption.
Fix code
```diff
    function mint(address account, uint256 amount) public onlyOwner {
-       _mint(account, amount);
+       if (amount > 0){
+         _mint(account, amount);
+       }  
    }
```


## [G-30] Do not perform depositRewards when totalReward is 0 to save gas


```solidity
80  protocolRewards.depositRewards{ value: totalReward }(
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L80

- With this change, the depositRewards function will only be called if the totalReward amount is greater than 0, allowing you to save gas by skipping the function call when there are no rewards to deposit.

```diff
    function _depositPurchaseRewards(
        uint256 paymentAmountWei,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {
        (RewardsSettings memory settings, uint256 totalReward) = computePurchaseRewards(paymentAmountWei);

        if (builderReferral == address(0)) builderReferral = revolutionRewardRecipient;

        if (deployer == address(0)) deployer = revolutionRewardRecipient;

        if (purchaseReferral == address(0)) purchaseReferral = revolutionRewardRecipient;
        
+       if (totalReward > 0) {
        protocolRewards.depositRewards{ value: totalReward }(
            builderReferral,
            settings.builderReferralReward,
            purchaseReferral,
            settings.purchaseReferralReward,
            deployer,
            settings.deployerReward,
            revolutionRewardRecipient,
            settings.revolutionReward
        );
+       }
        return totalReward;
    }
```

## [G-31] Do not perform deposit when _amount is 0 to save gas
- additional condition `_amount == 0` is added to the first if statement. If the `_amount` is zero, the function will revert with the error message `"Invalid amount."` This prevents the transfer from being executed when the `amount` is zero, saving gas and avoiding unnecessary operations.

```diff
File: packages/revolution/src/AuctionHouse.sol
    function _safeTransferETHWithFallback(address _to, uint256 _amount) private {
        // Ensure the contract has enough ETH to transfer
-       if (address(this).balance < _amount) revert("Insufficient balance");
+       if (address(this).balance < _amount || _amount == 0) revert("Invalid amount");

        // Used to store if the transfer succeeded
        bool success;

        assembly {
            // Transfer ETH to the recipient
            // Limit the call to 50,000 gas
            success := call(50000, _to, _amount, 0, 0, 0, 0)
        }

        // If the transfer failed:
        if (!success) {
            // Wrap as WETH
            IWETH(WETH).deposit{ value: _amount }();

            // Transfer WETH instead
            bool wethSuccess = IWETH(WETH).transfer(_to, _amount);

            // Ensure successful transfer
            if (!wethSuccess) revert("WETH transfer failed");
        }
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L419-L443


## [G-32] Refactor a modifier to call a local function instead of directly having the code in the modifier, saving bytecode size and thereby deployment cost 
Modifiers code is copied in all instances where it's used, increasing bytecode size. By doing a refractor to the internal function, one can reduce bytecode size significantly at the cost of one JUMP. Consider doing this only if you are constrained by bytecode size.

Before:
```solidity
modifier onlyOwner() {
		require(owner() == msg.sender, "Ownable: caller is not the owner");
		_;
}

```
After:
```solidity
modifier onlyOwner() {
		_checkOwner();
		_;
}

function _checkOwner() internal view virtual {
    require(owner() == msg.sender, "Ownable: caller is not the owner");
}

```


- #### By refactoring the modifiers in VerbsToken contract to call internal functions, you can reduce the bytecode size and deployment cost.

```solidity
File: packages/revolution/src/VerbsToken.sol
75  modifier whenMinterNotLocked() {
        require(!isMinterLocked, "Minter is locked");
        _;
    }

    /**
     * @notice Require that the CultureIndex has not been locked.
     */
83  modifier whenCultureIndexNotLocked() {
        require(!isCultureIndexLocked, "CultureIndex is locked");
        _;
    }

    /**
     * @notice Require that the descriptor has not been locked.
     */
91  modifier whenDescriptorNotLocked() {
        require(!isDescriptorLocked, "Descriptor is locked");
        _;
    }

    /**
     * @notice Require that the sender is the minter.
     */
99  modifier onlyMinter() {
        require(msg.sender == minter, "Sender is not the minter");
        _;
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L75-L102


```solidity
File: packages/revolution/src/MaxHeap.sol
41  modifier onlyAdmin() {
        require(msg.sender == admin, "Sender is not the admin");
        _;
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L41-L44