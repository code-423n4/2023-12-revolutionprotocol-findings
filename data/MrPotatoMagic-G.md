# Gas Optimizations Report

| ID     | Optimization                                                                                            |
|--------|---------------------------------------------------------------------------------------------------------|
| [G-01] | `MAX_NUM_CREATORS` check is not required when minting in VerbsToken.sol contract                        |
| [G-02] | Mappings not used externally/internally can be marked private                                           |
| [G-03] | No need to initialize variable `size` to 0                                                              |
| [G-04] | Use `left + 1` to calculate value of `right` in `maxHeapify()` to save gas                              |
| [G-05] | Place size check at the start of maxHeapify() to save gas on returning case                             |
| [G-06] | Cache `parent(current)` in insert() function to save gas                                                |
| [G-07] | else-if block in function updateValue() can be removed since newValue can never be less than oldValue   |
| [G-08] | `size > 0` check not required in function getMax()                                                      |
| [G-09] | Cache `msgValueRemaining - toPayTreasury` in buyToken() to save gas                                     |
| [G-10] | `creatorsAddress != address(0)` check not required in buyToken()                                        |
| [G-11] | Cache return value of _calculateTokenWeight() function to prevent SLOAD                                 |
| [G-12] | Cache `erc20VotingToken.totalSupply()` to save gas                                                      |
| [G-13] | Unnecessary for loop can be removed by shifting its statements into an existing for loop                |
| [G-14] | Return memory variable `pieceId` instead of storage variable `newPiece.pieceId` to save gas             |
| [G-15] | Calculate `creatorsShare` before `auctioneerPayment` in buyToken() to prevent unnecessary SUB operation |
| [G-16] | Remove `msgValue < computeTotalReward(msgValue` check from TokenEmitterRewards.sol contract             |
| [G-17] | Optimize `computeTotalReward()` and `computePurchaseRewards` into one function to save gas              |
| [G-18] | Calculation in computeTotalReward() can be simplified to save gas                                       |

**Optimizations have been focused on specific contracts/functions as requested by the sponsor in the README [here under Gas Reports](https://code4rena.com/audits/2023-12-revolution-protocol#toc-2-gas-reports).**

## [G-01] `MAX_NUM_CREATORS` check is not required when minting in VerbsToken.sol contract

The check below in the _mintTo() function can be removed since it is already checked when creating a piece in CultureIndex.sol (see [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L182)).
```solidity
File: VerbsToken.sol
288:         require(
289:             artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
290:             "Creator array must not be > MAX_NUM_CREATORS"
291:         );
```

## [G-02] Mappings not used externally/internally can be marked private

The below mappings from the [MaxHeap.sol contract](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L65) can be marked private since they're not used by any other contracts.
```solidity
File: MaxHeap.sol
66:     /// @notice Struct to represent an item in the heap by it's ID
67:     mapping(uint256 => uint256) public heap; 
68: 
69:     uint256 public size = 0;
70: 
71:     /// @notice Mapping to keep track of the value of an item in the heap
72:     mapping(uint256 => uint256) public valueMapping; 
73: 
74:     /// @notice Mapping to keep track of the position of an item in the heap
75:     mapping(uint256 => uint256) public positionMapping; 
```

## [G-03] No need to initialize variable `size` to 0

0 is the default value for an unsigned integer, thus setting it to 0 again is not required.
```solidity
File: MaxHeap.sol
69:     uint256 public size = 0;
```

## [G-04] Use `left + 1` to calculate value of `right` in `maxHeapify()` to save gas

The variable `right` can be re-written as 2 * pos + 1 + 1. Since we already know `left` is 2 * pos + 1, we can use left + 1 to save gas. This would mainly remove the MUL opcode (5 gas) and CALLDATALOAD (3 gas) and add just an MLOAD opcode (3 gas) instead. Thus saving a net of 5 gas per call.

Instead of this:
```solidity
File: MaxHeap.sol
099:     function maxHeapify(uint256 pos) internal {
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = 2 * pos + 2; //@audit Gas - Just use left + 1
```
Use this:
```solidity
File: MaxHeap.sol
099:     function maxHeapify(uint256 pos) internal {
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = left + 1;
```

## [G-05] Place size check at the start of maxHeapify() to save gas on returning case

The check on Line 107 can be moved to the start of the [maxHeapify()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94) function. This will save gas when the condition becomes true because the computations from Line 100 to 105 will not be executed for the returning case since we will return early.

Instead of this:
```solidity
File: MaxHeap.sol
099:     function maxHeapify(uint256 pos) internal {
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = 2 * pos + 2; 
102: 
103:         uint256 posValue = valueMapping[heap[pos]];
104:         uint256 leftValue = valueMapping[heap[left]];
105:         uint256 rightValue = valueMapping[heap[right]];
106: 
107:         if (pos >= (size / 2) && pos <= size) return;
109: 
111:         if (posValue < leftValue || posValue < rightValue) {
113:             
114:             if (leftValue > rightValue) { 
115:                 swap(pos, left);
116:                 maxHeapify(left);
117:             } else {
118:                 swap(pos, right);
119:                 maxHeapify(right);
120:             }
121:         }
122:     }
```
Use this (see line 99):
```solidity
File: MaxHeap.sol
098:     function maxHeapify(uint256 pos) internal {
099:         if (pos >= (size / 2) && pos <= size) return;
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = 2 * pos + 2; 
102: 
103:         uint256 posValue = valueMapping[heap[pos]];
104:         uint256 leftValue = valueMapping[heap[left]];
105:         uint256 rightValue = valueMapping[heap[right]];
106: 
109: 
111:         if (posValue < leftValue || posValue < rightValue) {
113:             
114:             if (leftValue > rightValue) { 
115:                 swap(pos, left);
116:                 maxHeapify(left);
117:             } else {
118:                 swap(pos, right);
119:                 maxHeapify(right);
120:             }
121:         }
122:     }
```

## [G-06] Cache `parent(current)` in insert() function to save gas

[Link to instance](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L126C1-L127C39)
[Link to another instance](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L147)

Caching the parent() function call will save an unnecessary JUMPDEST and internal operations in the function itself.

Instead of this:
```solidity
File: MaxHeap.sol
138:             swap(current, parent(current));
139:             current = parent(current);
```
Use this:
```solidity
File: MaxHeap.sol
137:             uint256 parentOfCurrent = parentCurrent(current);
138:             swap(current, parentOfCurrent);
139:             current = parentOfCurrent;
```

## [G-07] else-if block in function updateValue() can be removed since newValue can never be less than oldValue

The else-if block below is present in the [updateValue()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L150) function. This can be removed because votes made from function [_vote()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L322C1-L322C51) cannot be cancelled or decreased, thus this case is never true. This will save both function execution cost and deployment cost.

```solidity
File: MaxHeap.sol
164:         } else if (newValue < oldValue) maxHeapify(position);
```

## [G-08] `size > 0` check not required in function getMax()

The check `size > 0` is already implemented in the function topVotedPieceId() [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L487) (which is accessed from dropTopVotedPiece()) in CultureIndex.sol, thus implementing it in getMax() function in MaxHeap.sol is not required.

```solidity
File: MaxHeap.sol
188:         require(size > 0, "Heap is empty"); 
```

## [G-09] Cache `msgValueRemaining - toPayTreasury` in buyToken() to save gas

The same subtraction operation with the same variables is carried out in three different steps. Consider caching this value to save gas.

Instead of this:
```solidity
File: ERC20TokenEmitter.sol
178:         uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
179: 
180:         //Tokens to emit to creators
181:         int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
182:             ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
183:             : int(0);
```
Use this (see Line 177):
```solidity
File: ERC20TokenEmitter.sol
177:         uint256 cachedValue = msgValueRemaining - toPayTreasury;
178:         uint256 creatorDirectPayment = ((cachedValue) * entropyRateBps) / 10_000;
179: 
180:         //Tokens to emit to creators
181:         int totalTokensForCreators = ((cachedValue) - creatorDirectPayment) > 0
182:             ? getTokenQuoteForEther((cachedValue) - creatorDirectPayment)
183:             : int(0);
```

## [G-10] `creatorsAddress != address(0)` check not required in buyToken()

The check below is from the buyToken() function (see [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L201)). The second condition `creatorsAddress != address(0)` can be removed since `creatorsAddress` can never be the zero address. This is because function [setCreatorsAddress()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L310) implements this check already.

```solidity
File: ERC20TokenEmitter.sol
205:         if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
```

## [G-11] Cache return value of _calculateTokenWeight() function to prevent SLOAD

In the function createPiece() [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L226), the totalVotesSupply returned can be cached into a memory variable and then assigned to Line 229 and 237. This will prevent the unnecessary SLOAD of `newPiece.totalVoteSupply` on Line 237. 
```solidity
File: CultureIndex.sol
229:         newPiece.totalVotesSupply = _calculateVoteWeight( 
230:             erc20VotingToken.totalSupply(), 
231:             erc721VotingToken.totalSupply()
232:         );
233:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
234:         newPiece.metadata = metadata;
235:         newPiece.sponsor = msg.sender; 
236:         newPiece.creationBlock = block.number; 
237:         newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000; 
```

## [G-12] Cache `erc20VotingToken.totalSupply()` to save gas

The value of the call `erc20VotingToken.totalSupply()` is used on Line 230 and Line 233. This can save an unnecessary totalSupply() call on the erc20VotingToken if cached.
```solidity
File: CultureIndex.sol
229:         newPiece.totalVotesSupply = _calculateVoteWeight( 
230:             erc20VotingToken.totalSupply(),
231:             erc721VotingToken.totalSupply()
232:         );
233:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
```

## [G-13] Unnecessary for loop can be removed by shifting its statements into an existing for loop  

The for loop on Line 247 is not required since the for loop on Line 239 already loops through the same range [0, creatorArrayLength). Thus the for loop on Line 247 can be removed and the event emissions of PieceCreatorAdded() can be moved into the for loop on Line 239. This would save gas on both deployment and during function execution.
```solidity
File: CultureIndex.sol
239:         for (uint i; i < creatorArrayLength; i++) {
240:             newPiece.creators.push(creatorArray[i]);
241:         }
242: 
243:         emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
244: 
245:         // Emit an event for each creator
246:     
247:         for (uint i; i < creatorArrayLength; i++) {
248:             emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
249:         }
```

## [G-14] Return memory variable `pieceId` instead of storage variable `newPiece.pieceId` to save gas

Return memory variable `pieceId` instead of accessing value from storage and returning. This would replace the SLOAD (100 gas) with an MLOAD (3 gas) operation.
```solidity
File: CultureIndex.sol
250:         return newPiece.pieceId;
```

## [G-15] Calculate `creatorsShare` before `auctioneerPayment` in buyToken() to prevent unnecessary SUB operation

On Line 388, when calculating the `auctioneerPayment`, we find out the bps for the auctioneer by subtracting 10000 from creatorRateBps. This SUB operation will not be required if we calculate the `creatorsShare` on Line 391 before the `auctioneerPayment`. In this case the `creatorsShare` can just be calculated using `_auction.amount * creatorRateBps / 10000`. Following which the `auctioneerPayment` can just be calculated using `_auction.amount - creatorsShare` (as done by creatorsShare previously on Line 391). Through this we can see a redundant SUB operation can be removed which helps save gas.
```solidity
File: AuctionHouse.sol
388:                 uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;
389: 
390:                 //Total amount of ether going to creator
391:                 uint256 creatorsShare = _auction.amount - auctioneerPayment;
```

## [G-16] Remove `msgValue < computeTotalReward(msgValue` check from TokenEmitterRewards.sol contract

On Line 18, when passing msgValue as parameter to computeTotalReward(), we will always pass this check because computeTotalReward always returns 2.5% of the msgValue. Thus since msgValue can never be less than 2.5% of itself, the condition never evaluates to true and we never revert.
```solidity
File: TokenEmitterRewards.sol
12:     function _handleRewardsAndGetValueToSend(
13:         uint256 msgValue,
14:         address builderReferral,
15:         address purchaseReferral,
16:         address deployer
17:     ) internal returns (uint256) {
18:         if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();
19: 
20:         return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
21:     }
```

## [G-17] Optimize `computeTotalReward()` and `computePurchaseRewards` into one function to save gas

Both the functions below do the same computation, which is not required if the functions are combined into one. This will also prevent the rounding issue mentioned [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L37).

Instead of this:
```solidity
File: RewardSplits.sol
41:     function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
42:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
43: 
45:         return
46:             (paymentAmountWei * BUILDER_REWARD_BPS) /
47:             10_000 +
48:             (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
49:             10_000 +
50:             (paymentAmountWei * DEPLOYER_REWARD_BPS) /
51:             10_000 +
52:             (paymentAmountWei * REVOLUTION_REWARD_BPS) /
53:             10_000;
54:     }
55: 
56:     function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
57:         return (
58:             RewardsSettings({
59:                 builderReferralReward: (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000,
60:                 purchaseReferralReward: (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000,
61:                 deployerReward: (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000,
62:                 revolutionReward: (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000
63:             }),
64:             computeTotalReward(paymentAmountWei)
65:         );
66:     }
```
Use this:
```solidity
File: RewardSplits.sol
41:     function computeTotalPurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
42:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
43:         
44:         uint256 br = (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000;
45:         uint256 pr = (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000;
46:         uint256 dr = (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000;
47:         uint256 rr = (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000;
48:
49:         return (
50:             RewardsSettings({
51:                 br,
52:                 pr,
53:                 dr,
54:                 rr
55:             }),
56:             br + pr + dr + rr
57:         );
58:     }
```

## [G-18] Calculation in computeTotalReward() can be simplified to save gas

Currently the calculation is very repetitive as seen below and some similar operations occur internally. If we observe closely, each reward separated by `+` have paymentAmountWei and 10000 common in them.

Let us assume x = paymentAmountWei, y = 10000 and a,b,c,d = each of the reward bps types respectively.

Current equation = (x * a)/y + (x * b)/y + (x * c)/y + (x * d)/y 

Let's take x / y common,

Modified Equation = x/y * a + x/y * b + x/y * c + x/y * d

Further let's take x/y common from all terms,

Modified Equation = x/y * (a + b + c + d)

Now since we need to get rid of the rounding, we just multiply first instead of divide.

**Final Equation** = (x * (a + b + c + d)) / y

This final equation will save us alot of gas without compromising on any rounding issues.

Instead of this:
```solidity
File: RewardSplits.sol
41:     function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
42:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
43: 
44:         
45:         return
46:             (paymentAmountWei * BUILDER_REWARD_BPS) /
47:             10_000 +
48:             (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
49:             10_000 +
50:             (paymentAmountWei * DEPLOYER_REWARD_BPS) /
51:             10_000 +
52:             (paymentAmountWei * REVOLUTION_REWARD_BPS) /
53:             10_000;
54:     }
```
Use this:
```solidity
File: RewardSplits.sol
41:     function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
42:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
43: 
44:         
45:         return
46:             (paymentAmountWei * (BUILDER_REWARD_BPS + PURCHASE_REFERRAL_BPS + DEPLOYER_REWARD_BPS + REVOLUTION_REWARD_BPS)) / 10_000;
47:     }
```