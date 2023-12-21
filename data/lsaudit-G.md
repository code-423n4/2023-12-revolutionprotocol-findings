# [G-01] Use multiple of `if` conditions instead of one with multiple of ORs

[File: RewardsSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L30)
```
if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero")
```

should be changed to:

```
if (_protocolRewards == address(0)) revert("Invalid Address Zero");
if (_revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");
```

[File: RewardsSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41)
```
if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

should be changed to:

```
if (paymentAmountWei <= minPurchaseAmount) revert INVALID_ETH_AMOUNT();
if (paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```


# [G-02] `msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer)` in `TokenEmitterRewards.sol` can be unchecked

[File: TokenEmitterRewards.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18-L20)
```
 if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();

        return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
```

Line 18 defines that `msgValue` has to be greater or equal to  `computeTotalReward(msgValue)`.
When we look at `_depositPurchaseRewards()` implementation, we can spot that it returns `computePurchaseRewards()`:

[File: RewardSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L66)
```
function _depositPurchaseRewards(
        uint256 paymentAmountWei,
[...]
        (RewardsSettings memory settings, uint256 totalReward) = computePurchaseRewards(paymentAmountWei);
[...]
        return totalReward;
```

Now, when we check `computePurchaseRewards()` implementation, we can see that it returns `computeTotalReward()`.

[File: RewardSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L54)
```
 function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
        return (
            RewardsSettings({
              [...]
            }),
            computeTotalReward(paymentAmountWei)
        );
    }
```

Since Line 18 already states that `msgValue` has to be greater or equal to  `computeTotalReward(msgValue)`. (otherwise, it will revert with `INVALID_ETH_AMOUNT()`), we can be sure that `msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer)` won't underflow - thus it can be unchecked.




# [G-03] Change post-incrementing to pre-incrementing

[File: VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L294)
```
 uint256 verbId = _currentVerbId++;
```

can be changed to ` uint256 verbId = ++_currentVerbId;`

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L218)
```
 uint256 pieceId = _currentPieceId++;
```

can be changed to `uint256 pieceId = ++_currentPieceId;`



# [G-04] Move `require` statements on top of the function

[File: AuctionHouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L123-L133)
```
        __Pausable_init();
        __ReentrancyGuard_init();
        __Ownable_init(_initialOwner);

        _pause();

        require(
            _auctionParams.creatorRateBps >= _auctionParams.minCreatorRateBps,
            "Creator rate must be greater than or equal to the creator rate"
        );
```

Above `require` can be moved on top, above `__Pausable_init();`. If condition won't be met, function will revert immediately, instead of wasting gas on performing other operations.


[File: ERC20TokenEmitter.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L93-L96)
```
        __Pausable_init();
        __ReentrancyGuard_init();

        require(_treasury != address(0), "Invalid treasury address");
```

Above `require` can be moved on top, above `__Pausable_init();`. If condition won't be met, function will revert immediately, instead of wasting gas on performing other operations.

# [G-05] Function `createBid()` in `AuctionHouse.sol` can be optimized

We can merge to `if` condition into one. That way we can achieve two optimizations:

* Getting rid of `extended` variable declaration
* Removing one `if` condition

[File: AuctionHouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L191-L199)
```
		bool extended = _auction.endTime - block.timestamp < timeBuffer;
        if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;

        // Refund the last bidder, if applicable
        if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);

        emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);

        if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);
```

Variable `extended` is used only twice, but it's used in `if` condition. Since the second `if` emits only an event, we can merge two conditions into one and get rid of `extended` declaration:

```
        if (_auction.endTime - block.timestamp < timeBuffer) { 
        	auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
        	emit AuctionExtended(_auction.verbId, _auction.endTime);
        }

        // Refund the last bidder, if applicable
        if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);

        emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);

```


# [G-06] Use `require` statements which uses less gas first in `AuctionHouse.sol`

[File: AuctionHouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L217)
```
function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
        require(
            _creatorRateBps >= minCreatorRateBps,
            "Creator rate must be greater than or equal to minCreatorRateBps"
        );
        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
```

First condition reads state variable `minCreatorRateBps`, while the second condition reads constant `10 000`. The 2nd condition uses less gas which means it should be first.


[File: AuctionHouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L233)
```
 function setMinCreatorRateBps(uint256 _minCreatorRateBps) external onlyOwner {
        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
        require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");
```

First condition reads state variable `creatorRateBps`, while the second condition reads constant `10 000`. The 2nd condition uses less gas which means it should be first.

# [G-07] `_auction.amount - auctioneerPayment` in `AuctionHouse` can be unchecked

[File: AuctionHouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L365-L368)
```
 				uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;

                //Total amount of ether going to creator
                uint256 creatorsShare = _auction.amount - auctioneerPayment;
```

Since we firstly multiple `_auction.amount` by `10_000 - creatorRateBps` and then divide by `10_000`, we can be sure, that `auctioneerPayment <= _auction.amount`.
This proves, that `uint256 creatorsShare = _auction.amount - auctioneerPayment;` won't underflow and can be unchecked.


# [G-08] Do not calculate the `length` of array multiple of times in `ERC20TokenEmitter.sol`

The bot-race result reported that `addresses.length` should not be used inside loop and it should be cached in the local variable before line 209.
However `addresses.length` should be cached earlier. ``addresses.length`` is being used in line 162, which means that it should be cached before that line. That way, we won't be calculating the length of `addresses` two times:

[File: ](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L162-L209)
```
162:    require(addresses.length == basisPointSplits.length, "Parallel arrays required");

[...]

209:    for (uint256 i = 0; i < addresses.length; i++) {
```

should be changed to:

```
uint256 addessesLength = addresses.length;
require(addessesLength == basisPointSplits.length, "Parallel arrays required");
[...]
for (uint256 i = 0; i < addessesLength; i++) {
```


# [G-09] Calculation of `toPayTreasury` in `ERC20TokenEmitter.sol` can be unchecked

Below issue describes multiple of steps which proves that `toPayTreasury` can be unchecked.

* `10_000 - creatorRateBps` can be unchecked

[File: ERC20TokenEmitter.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L173)
```
uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;
```

[File: ERC20TokenEmitter.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L173)
```
uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;
```

Function `setCreatorRateBps()` sets `creatorRateBps` variable. According to its restriction: `require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");` (line 300), we can be sure, that `creatorRateBps <= 10_000`. This implies, that `10_000 - creatorRateBps` will never underflow thus can be unchecked.

* `msgValueRemaining * (10_000 - creatorRateBps)` can be unchecked

Since `msgValueRemaining` is returned by `_handleRewardsAndGetValueToSend()` - we can be sure, that its value is smaller than `msg.data`. Since the total ETH Supply is less than 121 000 000, we can safely assume, that `msgValueRemaining * (10_000 - creatorRateBps)` will never overflow `uint256` - thus it can be unchecked.

* `(msgValueRemaining * (10_000 - creatorRateBps)) / 10_000`

Division can be always unchecked


# [G-10] Modify `if` condition to avoid reading state variable twice in `ERC20TokenEmitter.sol`

[File: ERC20TokenEmitter.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L187-L188)
```
        emittedTokenWad += totalTokensForBuyers;
        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;
```

Current implementation reads `emittedTokenWad` (state variable) twice. Firstly, to add `totalTokensForBuyers`, then (when `totalTokensForCreators > 0`) to add `totalTokensForCreators`.
To reduce the number of reading of `emittedTokenWad`, we can rewrite above code to:

```
if (totalTokensForCreators > 0) {
        emittedTokenWad += totalTokensForBuyers + totalTokensForCreators;
}
else {
        emittedTokenWad += totalTokensForBuyers;
}
```

# [G-11] Add additional `address(0)` check in `initialize()` in `ERC20TokenEmitter.sol` to avoid checking for `address(0)` in `buyToken()`

Because of the `initilizer` modifier, function `initialize()` can be called only once. This function sets the `creatorsAddress` variable.
Our recommendation is to add additional condition in `initialize()`: `require(creatorsAddress != address(0), "Cannot be address(0)")`.
This will guarantee, that `creatorsAddress` won't be `address(0)`. Moreover, function `setCreatorsAddress()` which updates that variable also performs that check:

[File: ERC20TokenEmitter.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L309-L310)
```
function setCreatorsAddress(address _creatorsAddress) external override onlyOwner nonReentrant {
        require(_creatorsAddress != address(0), "Invalid address");
```

Those two conditions guarantee that `creatorsAddress` will never be `address(0)`. This implies, that condition in function `buyToken()`:

[File: ERC20TokenEmitter.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L201)
```
 if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
```

can be reduced to:

```
 if (totalTokensForCreators > 0) {
```

This implements a significant improvement. We add just one `address(0)` check in the `initialize()` (which can be called only once), and removed that check from `buyToken()` (which can be called multiple of times).

# [G-12] Do-while loops are more gas-efficient than for-loops

[File: ERC20TokenEMitter.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L209)
```
for (uint256 i = 0; i < addresses.length; i++) {
```

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L185)
```
for (uint i; i < creatorArrayLength; i++) {
```

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L236)
```
for (uint i; i < creatorArrayLength; i++) {
```

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L243)
```
for (uint i; i < creatorArrayLength; i++) {
```

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L355)
```
for (uint256 i; i < len; i++) {
```

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L403)
```
for (uint256 i; i < len; i++) {
```

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L407)
```
 for (uint256 i; i < len; i++) {
```

Above loop can be rewritten to `do-while` loop. Especially, that it implements just a simple iteration, which can easily be re-implemented to a do-while loop.


# [G-13] If statements should be performed before `require` in `CultureIndex.sol`

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L160-L168)
```
        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

        if (metadata.mediaType == MediaType.IMAGE)
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        else if (metadata.mediaType == MediaType.ANIMATION)
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        else if (metadata.mediaType == MediaType.TEXT)
            require(bytes(metadata.text).length > 0, "Text must be provided");
```

The first `require` statement verifies if `metadata.mediaType` is in a (1, 5) range.
Then, `if`/`else` condition verifies what MediaType it is. However, please notice, that when `metadata.mediaType` is `MediaType.IMAGE` or `MediaType.ANIMATION` or `MediaType.TEXT`, then `metadata.mediaType` is already in that range - thus checking that is redundant. Above code can be rewritten to:

```
        if (metadata.mediaType == MediaType.IMAGE)
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        else if (metadata.mediaType == MediaType.ANIMATION)
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        else if (metadata.mediaType == MediaType.TEXT)
            require(bytes(metadata.text).length > 0, "Text must be provided");
        else
            require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");
```

# [G-14] Merge two for-loops into one in `CultureIndex.sol`

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L236-L245)
```
        for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

        // Emit an event for each creator
        for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }
```
Those two loops are iterating over the same indexes. The first one adds new creators, the second one emits an event. This means, that it can be done in the same loop, instead of iterating twice.

```
 for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
```

# [G-15] Redundant condition in `_verifyVoteSignature()` in `CultureIndex.sol`

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L438-L441)
```
        if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();
```

`ecrecover` returns signer address or `address(0)` when signature is not valid.

Let's consider every possible scenario:

* `from: 0xAAA`, `recoveredAddress: 0xAAA` (signature VALID).
In that case, `if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();` won't revert, since `recoveredAddress` is not `address(0)` and `recoveredAddress == from`.

* `from: 0xAAA`, `recoveredAddress: 0x0` (signature INVALID).
In that case, `if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();` will revert, since `recoveredAddress == address(0)`.

* `from: 0x0`, `recoveredAddress: 0x0` (signature AMBIGUOUS).
In that case, `if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();` will revert, since `recoveredAddress == address(0)`.

These conditions fulfill all scenarios. This implies that ` if (from == address(0)) revert ADDRESS_ZERO();` is redundant and can be removed, since `if (recoveredAddress == address(0) || recoveredAddress != from)` already checks all possible conditions.

# [G-16] Post-decrement in `MaxHeap.sol` can be unchecked

[File: MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L157-L160)
```
        require(size > 0, "Heap is empty");

        uint256 popped = heap[0];
        heap[0] = heap[--size];
```

Because of the `require`, `size > 0`, thus we know that `heap[--size]` won't underflow and can be unchecked.

# [G-17] `parent()` calculation in `MaxHeap` can be optimized

[File: MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L78-L80)
```
 function parent(uint256 pos) private pure returns (uint256) {
        require(pos != 0, "Position should not be zero");
        return (pos - 1) / 2;
    }
```

There are 2 ways to re-implement `parent()` so it will use less gas.

1. Since `pos > 0`, `(pos - 1) / 2` can be unchecked (`pos - 1` won't underflow and division can be unchecked)
2. Division by two can be calculated by using right shifts (this was already reported in bot-race). However, please notice, that when right shift is being used, which is also equivalent to division by two - operation can still remain unchecked.


# [G-18] Use short-circuit in `require` statemens in `MaxHeap.sol`

Solidity uses short-circuiting when evaluating boolean expressions. E.g. for `A and B` expression, when `A == false`, `B` won't be evaluated (because since `A` is already `false`, `A AND B` also will be `false` no matter what value `B` has). This behavior suggests, that it's better to use expressions which uses less gas first.

[File: MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L102C15-L102C45)
```
if (pos >= (size / 2) && pos <= size) return;
```

`pos <= size` uses less gas than `pos >= (size / 2)` (`<=` operator vs `>=` and division by 2). This impies, that above line of code should be rewritten to:

```
if (pos <= size && pos >= (size / 2)) return;
```


# [G-19] Do not use double negation when not needed

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L311)
```
require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
```

Above code can be simplified to:

```
require(votes[pieceId][voter].voterAddress == address(0), "Already voted");
```

# [G-20] Redundant `address(0)` check in `VerbsToken.sol`

[File: VerbToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L140)
```
require(_initialOwner != address(0), "Initial owner cannot be zero address");
```

Above check is redundant and can be removed. `_initialOwner` is being passed to Open Zeppelin's ` __Ownable_init()`. This function calls `__Ownable_init_unchained()` which already performs `address(0)` check and reverts with `OwnableInvalidOwner()` when `_initialOwner == address(0)`.