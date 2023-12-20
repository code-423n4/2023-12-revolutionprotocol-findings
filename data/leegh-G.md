# Gas Optimizations
## [G-01] Elements are `push`ed into dynamic array one by one.
If the size of a dynamic array is known, it is more efficient to alloc space in advance and fill elements by index than to `push` each element one by one.
```solidity
File: packages/revolution/src/CultureIndex.sol

236:        for (uint i; i < creatorArrayLength; i++) {
237:            newPiece.creators.push(creatorArray[i]);
238:        }
```
[CultureIndex.sol#L236-L238](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/CultureIndex.sol#L236-L238)
```solidity
File: packages/revolution/src/VerbsToken.sol

306:            for (uint i = 0; i < artPiece.creators.length; i++) {
307:                newPiece.creators.push(artPiece.creators[i]);
308:            }
```
[VerbsToken.sol#L306-L308](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/VerbsToken.sol#L306-L308)

## [G-02] Use local variable instead of state variable.
If local variable has the same meaning and value as state variable, use local variable instead of state variable.
```solidity
File: packages/revolution/src/CultureIndex.sol

247:        return newPiece.pieceId; // @audit use local variable `pieceId`
```
[CultureIndex.sol#L247](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/CultureIndex.sol#L247)

## [G-03] Use `a == b` instead of `!(a != b)`.
```solidity
File: packages/revolution/src/CultureIndex.sol

311:        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
```
[CultureIndex.sol#L311](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/CultureIndex.sol#L311)

## [G-04] Unnecessary checks.
`artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS()` always holds, as it is guaranteed in function `CultureIndex.createPiece`. So we can remove `L282-L289` to save gas.
```solidity
File: packages/revolution/src/VerbsToken.sol

282:        ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();
283:
284:        // Check-Effects-Interactions Pattern
285:        // Perform all checks
286:        require(
287:            artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
288:            "Creator array must not be > MAX_NUM_CREATORS"
289:        );
```
[VerbsToken.sol#L282-L289](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/VerbsToken.sol#L282-L289)

## [G-05] Unnecessary assignment statement.
```solidity
File: packages/revolution/src/VerbsToken.sol

292:        try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
293:            artPiece = _artPiece; // @audit no need to set `artPiece`, use `_artPiece` directly
294:            uint256 verbId = _currentVerbId++;
295:
296:            ICultureIndex.ArtPiece storage newPiece = artPieces[verbId];
297:
298:            newPiece.pieceId = artPiece.pieceId;
299:            newPiece.metadata = artPiece.metadata;
300:            newPiece.isDropped = artPiece.isDropped;
301:            newPiece.sponsor = artPiece.sponsor;
302:            newPiece.totalERC20Supply = artPiece.totalERC20Supply;
303:            newPiece.quorumVotes = artPiece.quorumVotes;
304:            newPiece.totalVotesSupply = artPiece.totalVotesSupply;
```
[VerbsToken.sol#L292-L304](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/VerbsToken.sol#L292-L304)

## [G-06] Unnecessary checking for `_auction.amount > 0`.
If `_auction.bidder` is not zero (`L358`), then the `_auction.amount` is not zero either (`L363`). This is guaranteed in function `createBid`. So `L363` is not necessary and can be removed.
```solidity
File: packages/revolution/src/AuctionHouse.sol

358:            if (_auction.bidder == address(0))
359:                verbs.burn(_auction.verbId);
360:                //If someone has bid, transfer the Verb to the winning bidder
361:            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);
362:
363:            if (_auction.amount > 0) { // @audit unnecessary statement
```
[AuctionHouse.sol#L358-L363](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/AuctionHouse.sol#L358-L363)

## [G-07] Cache function result to avoid re-calling.
`verbs.getArtPieceById(_auction.verbId)` is called 3 times in `_settleAuction`. We can cache its result to save gas.
```solidity
File: packages/revolution/src/AuctionHouse.sol

370:                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
371:                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

385:                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
```
[AuctionHouse.sol#L370-L385](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/AuctionHouse.sol#L370-L385)

## [G-08] Identical arithmetic operations in loop.
`creatorsShare * entropyRateBps` are calculated in each iteration of the loop. As `creatorsShare` and `entropyRateBps` have nothing to do with the loop, we can move the multiplication out of the loop to save gas.
```solidity
File: packages/revolution/src/AuctionHouse.sol

384:                    for (uint256 i = 0; i < numCreators; i++) {
385:                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
386:                        vrgdaReceivers[i] = creator.creator;
387:                        vrgdaSplits[i] = creator.bps;
388:
389:                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
390:                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000); // @audit cache `creatorsShare * entropyRateBps`
391:                        ethPaidToCreators += paymentAmount;
392:
393:                        //Transfer creator's share to the creator
394:                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
395:                    }
```
[AuctionHouse.sol#L384-L395](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/AuctionHouse.sol#L384-L395)

## [G-09] Cache the intermediate results of arithmetic operations to avoid re-calculate them.
`(msgValueRemaining - toPayTreasury)` and `(msgValueRemaining - toPayTreasury) - creatorDirectPayment` are used multiple times in the code. Cache their results to save gas.
```solidity
File: packages/revolution/src/ERC20TokenEmitter.sol

177:        uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
178:        //Tokens to emit to creators
179:        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
180:            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
181:            : int(0);
```
[ERC20TokenEmitter.sol#L177-L181](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/ERC20TokenEmitter.sol#L177-L181)

## [G-10] Redundant arithmetic operations.
At the end of function `computePurchaseRewards`, the four kinds of rewards are calculated once again in `computeTotalReward` in the same way as `L57-L60`. We can just sum all the 4 rewards to get the total. Addition needs lower gas than multiplication.
```solidity
File: packages/protocol-rewards/src/abstract/RewardSplits.sol

54:    function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
55:        return (
56:            RewardsSettings({
57:                builderReferralReward: (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000,
58:                purchaseReferralReward: (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000,
59:                deployerReward: (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000,
60:                revolutionReward: (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000
61:            }),
62:            computeTotalReward(paymentAmountWei) // @audit sum the above 4 rewards
63:        );
64:    }
```
[RewardSplits.sol#L54-L64](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L54-L64)

## [G-11] Consider merging `if` statements with same conditions.
Merge `if` statements (`L192` and `L199`) if they share the same conditions to save one condition comparision.
```solidity
File: packages/revolution/src/AuctionHouse.sol

191:        bool extended = _atuction.endTime - block.timestamp < timeBuffer;
192:        if (extended) aucion.endTime = _auction.endTime = block.timestamp + timeBuffer; // @audit
193:
194:        // Refund the last bidder, if applicable
195:        if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);
196:
197:        emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);
198:
199:        if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime); // @audit
``` 
[AuctionHouse.sol#L191-L199](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/AuctionHouse.sol#L191-L199)