# REVOLUTION PROTOCOL GAS OPTIMIZATIONS


## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."




## [G-01] Avoid caching the length of calldata
Caching the length of calldata increases gas cost instead of reducing it

### Proof of Concept
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CacheCallDataLength {
    

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {
        uint256 len = numbers.length;

        for(uint i; i < len; ++i) {
            total += numbers[i];
        }

    }
}

```
```
test for test/CacheCallDataLength.t.sol:CacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1633)
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
contract NoCacheCallDataLength {
    

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {

        for(uint i; i < numbers.length; ++i) {
            total += numbers[i];
        }
        
    }
}
```
```
test for test/NoCacheCallDataLength.t.sol:NoCacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1628)
```


### 2 Instances

1. #### Reduce gas cost of `CultureIndex.validateCreatorsArray()` function by not caching the length of calldata.
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L180

```solidity
file: revolution/src/CultureIndex.sol

179:    function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
180:        uint256 creatorArrayLength = creatorArray.length;
181:        //Require that creatorArray is not more than MAX_NUM_CREATORS to prevent gas limit issues
182:        require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");
183:
184:        uint256 totalBps;
185:        for (uint i; i < creatorArrayLength; i++) {
186:            require(creatorArray[i].creator != address(0), "Invalid creator address");
187:            totalBps += creatorArray[i].bps;
188:        }
189:
190:        require(totalBps == 10_000, "Total BPS must sum up to 10,000");
191:
192:        return creatorArrayLength;
193:    }
```

```diff
 git diff
diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
index 88f3338..68be7b8 100644
--- a/packages/revolution/src/CultureIndex.sol
+++ b/packages/revolution/src/CultureIndex.sol
@@ -177,19 +177,18 @@ contract CultureIndex is
      * - The function will return the length of the `creatorArray`.
      */
     function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
-        uint256 creatorArrayLength = creatorArray.length;
         //Require that creatorArray is not more than MAX_NUM_CREATORS to prevent gas limit issues
         require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

         uint256 totalBps;
-        for (uint i; i < creatorArrayLength; i++) {
+        for (uint i; i < creatorArray.length; i++) {
             require(creatorArray[i].creator != address(0), "Invalid creator address");
             totalBps += creatorArray[i].bps;
         }

         require(totalBps == 10_000, "Total BPS must sum up to 10,000");

-        return creatorArrayLength;
+        return creatorArray.length;
     }

     /**
```

2. #### Reduce gas cost of `CultureIndex._voteForMany()` function by not caching the length of calldata.
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L354

```solidity
file: revolution/src/CultureIndex.sol

353:    function _voteForMany(uint256[] calldata pieceIds, address from) internal {
354:        uint256 len = pieceIds.length;
355:        for (uint256 i; i < len; i++) {
356:            _vote(pieceIds[i], from);
357:        }
358:    }
```

```diff
diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
index 88f3338..4dd3186 100644
--- a/packages/revolution/src/CultureIndex.sol
+++ b/packages/revolution/src/CultureIndex.sol
@@ -351,8 +351,7 @@ contract CultureIndex is
      * Emits a series of VoteCast event upon successful execution.
      */
     function _voteForMany(uint256[] calldata pieceIds, address from) internal {
-        uint256 len = pieceIds.length;
-        for (uint256 i; i < len; i++) {
+        for (uint256 i; i < pieceIds.length; i++) {
             _vote(pieceIds[i], from);
         }
     }
```


## [G-02] We can optimize the `VerbsToken.setMinter()` function

The `VerbsToken.setMinter()` function has a modifier `whenMinterNotLocked` which reads  state variable `isMinterLocked`. If the check does not pass we have a revert. Using this modifier in our function means we run the check first and revert if the check does not pass. If the check passes, the `VerbsToken.setMinter()` function has one more check `require(_minter != address(0), "Minter cannot be zero address")` which basically checks that the parameter `_minter` address is not zero. This is way cheaper compared to the check inside the modifier if this check does not pass, it means we would also revert, now suppose the first check(modifier) passes but the non zero fails, the gas consumed inside the modifier doing the state read be wasted. We can do this check more efficiently by prioritizing cheaper checks first but for this to work, we would need to inline our modifier as shown in the diff below:

- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L209-#L214
```solidity
file: revolution/src/VerbsToken.sol

209:    function setMinter(address _minter) external override onlyOwner nonReentrant whenMinterNotLocked {
210:        require(_minter != address(0), "Minter cannot be zero address");
211:        minter = _minter;
212:
213:        emit MinterUpdated(_minter);
214:    }
```

```diff
diff --git a/packages/revolution/src/VerbsToken.sol b/packages/revolution/src/VerbsToken.sol
index 7bd9527..6726c4d 100644
--- a/packages/revolution/src/VerbsToken.sol
+++ b/packages/revolution/src/VerbsToken.sol
@@ -206,8 +206,9 @@ contract VerbsToken is
      * @notice Set the token minter.
      * @dev Only callable by the owner when not locked.
      */
-    function setMinter(address _minter) external override onlyOwner nonReentrant whenMinterNotLocked {
+    function setMinter(address _minter) external override onlyOwner nonReentrant {
         require(_minter != address(0), "Minter cannot be zero address");
+        require(!isMinterLocked, "Minter is locked");
         minter = _minter;

         emit MinterUpdated(_minter);
```
```
Estimated gas saved: 2100 gas units
```



## [G-03] Refactor `ERC20TokenEmitter.buyToken()` function to avoid making the same checks on every loop iteration.
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L210

Having to perform the same check `if (totalTokensForBuyers > 0)` on every iteration of the loop is gas consuming, redundant and unnecessary since the check is not dependent on the loop iteration. We can reduce the gas cost of the `ERC20TokenEmitter.buyToken()` function if we move the check outside of the loop and then nest the loop within the check (the if statement). The diff below shows how the code could be refactored:

```solidity
file: revolution/src/ERC20TokenEmitter.sol

152:    function buyToken(
153:        address[] calldata addresses,
154:        uint[] calldata basisPointSplits,
155:        ProtocolRewardAddresses calldata protocolRewardsRecipients
156:    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
.
.
.
183:        // Tokens to emit to buyers
184:        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);
185:
186:        //Transfer ETH to treasury and update emitted
187:        emittedTokenWad += totalTokensForBuyers;
188:        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;
189:
190:        //Deposit funds to treasury
191:        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
192:        require(success, "Transfer failed.");
193:
194:        //Transfer ETH to creators
195:        if (creatorDirectPayment > 0) {
196:            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
197:            require(success, "Transfer failed.");
198:        }
199:
200:        //Mint tokens for creators
201:        if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
202:            _mint(creatorsAddress, uint256(totalTokensForCreators));
203:        }
204:
205:        uint256 bpsSum = 0;
206:
207:        //Mint tokens to buyers
208:
209:        for (uint256 i = 0; i < addresses.length; i++) {
210:            if (totalTokensForBuyers > 0) {   @audit same check on every loop iterations
211:                // transfer tokens to address
212:                _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
213:            }
214:            bpsSum += basisPointSplits[i];
215:        }
.
.
.
230:    }
```

```diff
diff --git a/packages/revolution/src/ERC20TokenEmitter.sol b/packages/revolution/src/ERC20TokenEmitter.sol
index e6f7d46..5c3b6b2 100644
--- a/packages/revolution/src/ERC20TokenEmitter.sol
+++ b/packages/revolution/src/ERC20TokenEmitter.sol
@@ -206,12 +206,13 @@ contract ERC20TokenEmitter is

         //Mint tokens to buyers

-        for (uint256 i = 0; i < addresses.length; i++) {
-            if (totalTokensForBuyers > 0) {
+        if (totalTokensForBuyers > 0) {
+            for (uint256 i = 0; i < addresses.length; i++) {
                 // transfer tokens to address
                 _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
+                bpsSum += basisPointSplits[i];
             }
-            bpsSum += basisPointSplits[i];
+
         }

         require(bpsSum == 10_000, "bps must add up to 10_000");
```


## [G-04] Computations should be memoized rather than having to re-compute them
In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive computations and returning the cached result when the same inputs occur again.

### 1 Instance
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L177-#L180

Rather than having to perform the calculations `msgValueRemaining - toPayTreasury` and `((msgValueRemaining - toPayTreasury) - creatorDirectPayment)` multiple times in the function we should memoize the result of the calculations (i.e cache the result of the calculation the first time) then use the cache result subsequently rather than having to re-perform the calculations. The diff below shows how the code could be refcatored: 

```solidity
file: revolution/src/ERC20TokenEmitter.sol

152:    function buyToken(
153:        address[] calldata addresses,
154:        uint[] calldata basisPointSplits,
155:        ProtocolRewardAddresses calldata protocolRewardsRecipients
156:    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
.
.
.
172:        //Share of purchase amount to send to treasury
173:        uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;
174:
175:        //Share of purchase amount to reserve for creators
176:        //Ether directly sent to creators
177:        uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
178:        //Tokens to emit to creators
179:        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
180:            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
181:            : int(0);
.
.
.
230:    }
```

```
diff --git a/packages/revolution/src/ERC20TokenEmitter.sol b/packages/revolution/src/ERC20TokenEmitter.sol
index e6f7d46..5fed0a4 100644
--- a/packages/revolution/src/ERC20TokenEmitter.sol
+++ b/packages/revolution/src/ERC20TokenEmitter.sol
@@ -174,10 +174,12 @@ contract ERC20TokenEmitter is

         //Share of purchase amount to reserve for creators
         //Ether directly sent to creators
-        uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
+        uint256 _creatorDirectPayment = msgValueRemaining - toPayTreasury;
+        uint256 creatorDirectPayment = ((_creatorDirectPayment) * entropyRateBps) / 10_000;
         //Tokens to emit to creators
-        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
-            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
+        uint256 _totalTokensForCreators = _creatorDirectPayment - creatorDirectPayment;
+        int totalTokensForCreators = _totalTokensForCreators > 0
+            ? getTokenQuoteForEther(_totalTokensForCreators)
             : int(0);
```



## [G-05] Refactor `Erc20TokenEmitter.buyTokens()` function to avoid storage read and write in some scenarios
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L184-#L187

In the `Erc20TokenEmitter.buyTokens()` function as shown below the ternary statement on [Line 184](- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L184-#L187) show that in some scenarios the value of `totalTokensForBuyers` could be 0 therefore in such scenarios we should avoid incrementing the 
value of `emittedTokenWad` by 0 as it makes no sense as this would cost `2100` for `SLOAD` and `2900` for `SSTORE` rather we could refactor the function such that the value of `emittedTokenWad` is only incremented when the value of `totalTokensForBuyers` is not zero. The diff below shows how we could refactor the code:

```solidity
file: revolution/src/ERC20TokenEmitter.sol

152:    function buyToken(
153:        address[] calldata addresses,
154:        uint[] calldata basisPointSplits,
155:        ProtocolRewardAddresses calldata protocolRewardsRecipients
156:    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
.
.
.
183:        // Tokens to emit to buyers
184:        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);
185:
186:        //Transfer ETH to treasury and update emitted
187:        emittedTokenWad += totalTokensForBuyers;
188:        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;
.
.
.
230:    }
```

```diff
diff --git a/packages/revolution/src/ERC20TokenEmitter.sol b/packages/revolution/src/ERC20TokenEmitter.sol
index e6f7d46..2e66b8e 100644
--- a/packages/revolution/src/ERC20TokenEmitter.sol
+++ b/packages/revolution/src/ERC20TokenEmitter.sol
@@ -181,10 +181,17 @@ contract ERC20TokenEmitter is
             : int(0);

         // Tokens to emit to buyers
-        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);
-
-        //Transfer ETH to treasury and update emitted
-        emittedTokenWad += totalTokensForBuyers;
+        int totalTokensForBuyers;
+
+        if (toPayTreasury > 0) {
+            totalTokensForBuyers = getTokenQuoteForEther(toPayTreasury);
+            //Transfer ETH to treasury and update emitted
+            emittedTokenWad += totalTokensForBuyers;
+        }else{
+            totalTokensForBuyers = int(0);
+        }
+
+
         if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

```
```
Estimated Gas saved: 5000 gas units
```


## [G-06] Refactor functions to avoid unnecessary external calls

### Instances

1. #### Refactor `AuctionHouse._settleAuction()` function to avoid unnecessary external calls
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L336-#L371

In the `AuctionHouse._settleAuction()` function as shown below rather than having to perform the external call `verbs.getArtPieceById(_auction.verbId)` on multiple occasions in the function (even in a loop) we can make the external call once, cache the returned struct in to a memory variable then use then read from the memory variable subsequently. In implementing this change we would save above `100` gas unit for every subsequent external call.

```solidity
file: revolution/src/AuctionHouse.sol

336:    function _settleAuction() internal {
.
.
.
363:            if (_auction.amount > 0) {
364:                // Ether going to owner of the auction
365:                uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;
366:
367:                //Total amount of ether going to creator
368:                uint256 creatorsShare = _auction.amount - auctioneerPayment;
369:
370:                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;    @audit external call unncessary
371:                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;    @audit external call unncessary
372:
373:                //Build arrays for erc20TokenEmitter.buyToken
374:                uint256[] memory vrgdaSplits = new uint256[](numCreators);
375:                address[] memory vrgdaReceivers = new address[](numCreators);
376:
377:                //Transfer auction amount to the DAO treasury
378:                _safeTransferETHWithFallback(owner(), auctioneerPayment);
379:
380:                uint256 ethPaidToCreators = 0;
381:
382:                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
383:                if (creatorsShare > 0 && entropyRateBps > 0) {
384:                    for (uint256 i = 0; i < numCreators; i++) {
385:                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];    @audit external call unncessary
386:                        vrgdaReceivers[i] = creator.creator;
387:                        vrgdaSplits[i] = creator.bps;
388:
389:                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
390:                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
391:                        ethPaidToCreators += paymentAmount;
392:
393:                        //Transfer creator's share to the creator
394:                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
395:                    }
396:                }
397:
.
.
.
414    }
```

```diff
diff --git a/packages/revolution/src/AuctionHouse.sol b/packages/revolution/src/AuctionHouse.sol
index 55e55a1..5a4fc36 100644
--- a/packages/revolution/src/AuctionHouse.sol
+++ b/packages/revolution/src/AuctionHouse.sol
@@ -367,8 +367,10 @@ contract AuctionHouse is
                 //Total amount of ether going to creator
                 uint256 creatorsShare = _auction.amount - auctioneerPayment;

-                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
-                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
+                ICultureIndex.ArtPiece memory artPiece = verbs.getArtPieceById(_auction.verbId);
+
+                uint256 numCreators = artPiece.creators.length;
+                address deployer = artPiece.sponsor;

                 //Build arrays for erc20TokenEmitter.buyToken
                 uint256[] memory vrgdaSplits = new uint256[](numCreators);
@@ -382,7 +384,7 @@ contract AuctionHouse is
                 //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                 if (creatorsShare > 0 && entropyRateBps > 0) {
                     for (uint256 i = 0; i < numCreators; i++) {
-                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
+                        ICultureIndex.CreatorBps memory creator = artPiece.creators[i];
                         vrgdaReceivers[i] = creator.creator;
                         vrgdaSplits[i] = creator.bps;
```

2. #### Refactor `CultureIndex.createPiece()` function to avoid unnecessary external calls
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L227&&#L230

In the `CultureIndex.createPiece()` function as shown below rather than having to perform the external call `erc20VotingToken.totalSupply()` on twice we can make the external call once, cache the returned value into a stack variable then use then read from the stack variable subsequently. In implementing this change we would save above `100` gas unit for second external call.

```solidity
file: revolution/src/CultureIndex.sol

209    function createPiece(
210:        ArtPieceMetadata calldata metadata,
211:        CreatorBps[] calldata creatorArray
212:    ) public returns (uint256) {
.
.
.
225:        newPiece.pieceId = pieceId;
226:        newPiece.totalVotesSupply = _calculateVoteWeight(
227:            erc20VotingToken.totalSupply(),
228:            erc721VotingToken.totalSupply()
229:        );
230:        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
231:        newPiece.metadata = metadata;
232:        newPiece.sponsor = msg.sender;
233:        newPiece.creationBlock = block.number;
234:        newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
.
.
.
248:    }
```

```diff
diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
index 88f3338..7e7079a 100644
--- a/packages/revolution/src/CultureIndex.sol
+++ b/packages/revolution/src/CultureIndex.sol
@@ -221,13 +221,13 @@ contract CultureIndex is
         maxHeap.insert(pieceId, 0);

         ArtPiece storage newPiece = pieces[pieceId];
-
+        uint256 _erc20VotingTokenTotalSupply = erc20VotingToken.totalSupply();
         newPiece.pieceId = pieceId;
         newPiece.totalVotesSupply = _calculateVoteWeight(
-            erc20VotingToken.totalSupply(),
+            _erc20VotingTokenTotalSupply,
             erc721VotingToken.totalSupply()
         );
-        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
+        newPiece.totalERC20Supply = _erc20VotingTokenTotalSupply;
         newPiece.metadata = metadata;
         newPiece.sponsor = msg.sender;
         newPiece.creationBlock = block.number;
```
```
Estimated Gas saved: 97 gas units
```




## [G-07] Refactor functions to avoid unnecessary SLOAD

### Instances
1. #### Refactor `CultureIndex.createPiece()` to avoid unnecessary SLOAD
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L226-#L240

In the `CultureIndex.createPiece()` function as shown below we can cache the calculation `_calculateVoteWeight(erc20VotingToken.totalSupply(), erc721VotingToken.totalSupply())` in to a stack variable, then assign the stack variable to `newPiece.totalVotesSupply` then use the stack variable in place of `newPiece.totalVotesSupply` subsequently. Also we can cache the calculation `(quorumVotesBPS * newPiece.totalVotesSupply) / 10_000` into a stack variable then assign the stack variable to `newPiece.quorumVotes` then use the stack variable in place of `newPiece.quorumVotes` subsequently.

```solidity
file: revolution/src/CultureIndex.sol

209:    function createPiece(
210:        ArtPieceMetadata calldata metadata,
211:        CreatorBps[] calldata creatorArray
212:    ) public returns (uint256) {
213:        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
214:
215:        // Validate the media type and associated data
216:        validateMediaType(metadata);
217:
218:        uint256 pieceId = _currentPieceId++;
219:
220:        /// @dev Insert the new piece into the max heap
221:        maxHeap.insert(pieceId, 0);
222:
223:        ArtPiece storage newPiece = pieces[pieceId];
224:
225:        newPiece.pieceId = pieceId;
226:        newPiece.totalVotesSupply = _calculateVoteWeight(
227:            erc20VotingToken.totalSupply(),
228:            erc721VotingToken.totalSupply()
229:        );
230:        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
231:        newPiece.metadata = metadata;
232:        newPiece.sponsor = msg.sender;
233:        newPiece.creationBlock = block.number;
234:        newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
235:
236:        for (uint i; i < creatorArrayLength; i++) {
237:            newPiece.creators.push(creatorArray[i]);
238:        }
239
240:        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
241:
242:        // Emit an event for each creator
243:        for (uint i; i < creatorArrayLength; i++) {
244:            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
245:        }
246:
247:        return newPiece.pieceId;
248:    }
```

```diff
diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
index 88f3338..c760096 100644
--- a/packages/revolution/src/CultureIndex.sol
+++ b/packages/revolution/src/CultureIndex.sol
@@ -222,22 +222,26 @@ contract CultureIndex is

         ArtPiece storage newPiece = pieces[pieceId];

-        newPiece.pieceId = pieceId;
-        newPiece.totalVotesSupply = _calculateVoteWeight(
+        uint256 _voteWeight = _calculateVoteWeight(
             erc20VotingToken.totalSupply(),
             erc721VotingToken.totalSupply()
         );
+
+        uint256 _quorumVotes = (quorumVotesBPS * _voteWeight) / 10_000;
+
+        newPiece.pieceId = pieceId;
+        newPiece.totalVotesSupply = _voteWeight;
         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
         newPiece.metadata = metadata;
         newPiece.sponsor = msg.sender;
         newPiece.creationBlock = block.number;
-        newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
+        newPiece.quorumVotes = _quorumVotes;

         for (uint i; i < creatorArrayLength; i++) {
             newPiece.creators.push(creatorArray[i]);
         }

-        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
+        emit PieceCreated(pieceId, msg.sender, metadata, _quorumVotes, _voteWeight);

         // Emit an event for each creator
         for (uint i; i < creatorArrayLength; i++) {
```


2. #### Refactor `ERC20TokenEmitter.buyToken()` to avoid unnecessary SLOAD
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L158

In the `ERC20TokenEmitter.buyToken()` we can avoid unnecessary `SLOAD` by caching the `creatorsAddress` and `treasury` state variables into stack variables and using the stack variables in place the state variables for subsequent reads.

```solidity
file: revolution/src/ERC20TokenEmitter.sol

152:    function buyToken(
153:        address[] calldata addresses,
154:        uint[] calldata basisPointSplits,
155:        ProtocolRewardAddresses calldata protocolRewardsRecipients
156:    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
157:        //prevent treasury from paying itself
158:        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");
159:
160:        require(msg.value > 0, "Must send ether");
161:        // ensure the same number of addresses and bps
162:        require(addresses.length == basisPointSplits.length, "Parallel arrays required");
163:
164:        // Get value left after protocol rewards
165:        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
166:            msg.value,
167:            protocolRewardsRecipients.builder,
168:            protocolRewardsRecipients.purchaseReferral,
169:            protocolRewardsRecipients.deployer
170:        );
171:
172:        //Share of purchase amount to send to treasury
173:        uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;
174:
175:        //Share of purchase amount to reserve for creators
176:        //Ether directly sent to creators
177:        uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
178:        //Tokens to emit to creators
179:        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
180:            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
181:            : int(0);
182:
183:        // Tokens to emit to buyers
184:        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);
185:
186:        //Transfer ETH to treasury and update emitted
187:        emittedTokenWad += totalTokensForBuyers;
188:        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;
189:
190:        //Deposit funds to treasury
191:        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
192:        require(success, "Transfer failed.");
193:
194:        //Transfer ETH to creators
195:        if (creatorDirectPayment > 0) {
196:            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
197:            require(success, "Transfer failed.");
198:        }
199:
200:        //Mint tokens for creators
201:        if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
202:            _mint(creatorsAddress, uint256(totalTokensForCreators));
203:        }
204:
205        uint256 bpsSum = 0;
206:
207:        //Mint tokens to buyers
208:
209        for (uint256 i = 0; i < addresses.length; i++) {
210:            if (totalTokensForBuyers > 0) {
211:                // transfer tokens to address
212:                _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
213:            }
214:            bpsSum += basisPointSplits[i];
215:        }
216:
217:        require(bpsSum == 10_000, "bps must add up to 10_000");
218:
219:        emit PurchaseFinalized(
220:            msg.sender,
221:            msg.value,
222:            toPayTreasury,
223:            msg.value - msgValueRemaining,
224:            uint256(totalTokensForBuyers),
225:            uint256(totalTokensForCreators),
226:            creatorDirectPayment
227:        );
228:
229:        return uint256(totalTokensForBuyers);
230    }
```

```diff
diff --git a/packages/revolution/src/ERC20TokenEmitter.sol b/packages/revolution/src/ERC20TokenEmitter.sol
index e6f7d46..2c64cbc 100644
--- a/packages/revolution/src/ERC20TokenEmitter.sol
+++ b/packages/revolution/src/ERC20TokenEmitter.sol
@@ -155,7 +155,9 @@ contract ERC20TokenEmitter is
         ProtocolRewardAddresses calldata protocolRewardsRecipients
     ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
         //prevent treasury from paying itself
-        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");
+        address _treasury = treasury;
+        address _creatorsAddress = creatorsAddress;
+        require(msg.sender != _treasury && msg.sender != _creatorsAddress, "Funds recipient cannot buy tokens");

         require(msg.value > 0, "Must send ether");
         // ensure the same number of addresses and bps
@@ -188,18 +190,18 @@ contract ERC20TokenEmitter is
         if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

         //Deposit funds to treasury
-        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
+        (bool success, ) = _treasury.call{ value: toPayTreasury }(new bytes(0));
         require(success, "Transfer failed.");

         //Transfer ETH to creators
         if (creatorDirectPayment > 0) {
-            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
+            (success, ) = _creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
             require(success, "Transfer failed.");
         }

         //Mint tokens for creators
-        if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
-            _mint(creatorsAddress, uint256(totalTokensForCreators));
+        if (totalTokensForCreators > 0 && _creatorsAddress != address(0)) {
+            _mint(_creatorsAddress, uint256(totalTokensForCreators));
         }

         uint256 bpsSum = 0;
```


3. #### Refactor `AuctionHouse._settleAuction()` to avoid unnecessary SLOAD
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L355
```solidity
file: revolution/src/AuctionHouse.sol

336:    function _settleAuction() internal {
337:        IAuctionHouse.Auction memory _auction = auction;
338:
339:        require(_auction.startTime != 0, "Auction hasn't begun");
340:        require(!_auction.settled, "Auction has already been settled");
341:        //slither-disable-next-line timestamp
342:        require(block.timestamp >= _auction.endTime, "Auction hasn't completed");
343:
344:        auction.settled = true;
345:
346:        uint256 creatorTokensEmitted = 0;
347:        // Check if contract balance is greater than reserve price
348:        if (address(this).balance < reservePrice) {
349:            // If contract balance is less than reserve price, refund to the last bidder
350:            if (_auction.bidder != address(0)) {
351:                _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
352:            }
353:
354:            // And then burn the Noun
355:            verbs.burn(_auction.verbId);
356:        } else {
357:            //If no one has bid, burn the Verb
358:            if (_auction.bidder == address(0))
359:                verbs.burn(_auction.verbId);
360:                //If someone has bid, transfer the Verb to the winning bidder
361:            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);
362:
363:            if (_auction.amount > 0) {
364:                // Ether going to owner of the auction
365:                uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;
366:
367:                //Total amount of ether going to creator
368:                uint256 creatorsShare = _auction.amount - auctioneerPayment;
369:
370:                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
371:                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
372:
373:                //Build arrays for erc20TokenEmitter.buyToken
374:                uint256[] memory vrgdaSplits = new uint256[](numCreators);
375:                address[] memory vrgdaReceivers = new address[](numCreators);
376:
377:                //Transfer auction amount to the DAO treasury
378:                _safeTransferETHWithFallback(owner(), auctioneerPayment);
379:
380:                uint256 ethPaidToCreators = 0;
381:
382:                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
383:                if (creatorsShare > 0 && entropyRateBps > 0) {
384:                    for (uint256 i = 0; i < numCreators; i++) {
385:                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
386:                        vrgdaReceivers[i] = creator.creator;
387:                        vrgdaSplits[i] = creator.bps;
388:
389:                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
390:                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
391:                        ethPaidToCreators += paymentAmount;
392:
393:                        //Transfer creator's share to the creator
394:                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
395:                    }
396:                }
397:
398:                //Buy token from ERC20TokenEmitter for all the creators
399:                if (creatorsShare > ethPaidToCreators) {
400:                    creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: creatorsShare - ethPaidToCreators }(
401:                        vrgdaReceivers,
402:                        vrgdaSplits,
403:                        IERC20TokenEmitter.ProtocolRewardAddresses({
404:                            builder: address(0),
405:                            purchaseReferral: address(0),
406:                            deployer: deployer
407:                        })
408:                    );
409:                }
410:            }
411:        }
412:
413:        emit AuctionSettled(_auction.verbId, _auction.bidder, _auction.amount, creatorTokensEmitted);
414:    }
```

```diff

```

4. #### Refactor `VerbsToken._mintTo()` to avoid unnecessary SLOAD
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L282

In the `VerbsToken._mintTo()` we can avoid unnecessary `SLOAD` by caching the `cultureIndex` state variables into stack variables and using the stack variables in place the state variables for subsequent reads.

```solidity
file: revolution/src/VerbsToken.sol

281:    function _mintTo(address to) internal returns (uint256) {
282:        ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();
283:
284:        // Check-Effects-Interactions Pattern
285:        // Perform all checks
286:        require(
287:            artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
288:            "Creator array must not be > MAX_NUM_CREATORS"
289:        );
290:
291:        // Use try/catch to handle potential failure
292:        try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
293:            artPiece = _artPiece;
294:            uint256 verbId = _currentVerbId++;
295:
.
.
.
319:        }
```

```diff
diff --git a/packages/revolution/src/VerbsToken.sol b/packages/revolution/src/VerbsToken.sol
index 7bd9527..e4b139f 100644
--- a/packages/revolution/src/VerbsToken.sol
+++ b/packages/revolution/src/VerbsToken.sol
@@ -279,17 +279,18 @@ contract VerbsToken is
      * @notice Mint a Verb with `verbId` to the provided `to` address. Pulls the top voted art piece from the CultureIndex.
      */
     function _mintTo(address to) internal returns (uint256) {
-        ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();
+        ICultureIndex _cultureIndex = cultureIndex;
+        ICultureIndex.ArtPiece memory artPiece = _cultureIndex.getTopVotedPiece();

         // Check-Effects-Interactions Pattern
         // Perform all checks
         require(
-            artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
+            artPiece.creators.length <= _cultureIndex.MAX_NUM_CREATORS(),
             "Creator array must not be > MAX_NUM_CREATORS"
         );

         // Use try/catch to handle potential failure
-        try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
+        try _cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
             artPiece = _artPiece;
             uint256 verbId = _currentVerbId++;
```
```
Estimated gas saved: 194 gas units
```




## [G-08] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

### Instances

1. #### Use `pieceId` in place of `newPiece.pieceId`
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L225

We can save 1 `SLOAD` `2100` gas units if we use `pieceId` in place of `newPiece.pieceId` since their values are equal 

```solidity
file: 

209:    function createPiece(
210:        ArtPieceMetadata calldata metadata,
211:        CreatorBps[] calldata creatorArray
212:    ) public returns (uint256) {
213:        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
214:
215:        // Validate the media type and associated data
216:        validateMediaType(metadata);
217:
218:        uint256 pieceId = _currentPieceId++;
219:
220:        /// @dev Insert the new piece into the max heap
221:        maxHeap.insert(pieceId, 0);
222:
223:        ArtPiece storage newPiece = pieces[pieceId];
224:
225:        newPiece.pieceId = pieceId;
226:        newPiece.totalVotesSupply = _calculateVoteWeight(
227:            erc20VotingToken.totalSupply(),
228:            erc721VotingToken.totalSupply()
229:        );
230:        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
231:        newPiece.metadata = metadata;
232:        newPiece.sponsor = msg.sender;
233:        newPiece.creationBlock = block.number;
234:        newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
235:
236:        for (uint i; i < creatorArrayLength; i++) {
237:            newPiece.creators.push(creatorArray[i]);
238:        }
239:
240:        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
241:
242:        // Emit an event for each creator
243:        for (uint i; i < creatorArrayLength; i++) {
244:            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
245:        }
246:
247:        return newPiece.pieceId;    @audit use pieceId to avoid SLOAD
248:    }
```

```diff
diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
index 88f3338..a6b0034 100644
--- a/packages/revolution/src/CultureIndex.sol
+++ b/packages/revolution/src/CultureIndex.sol
@@ -221,13 +221,13 @@ contract CultureIndex is
@@ -244,7 +244,7 @@ contract CultureIndex is
             emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
         }

-        return newPiece.pieceId;
+        return pieceId;
     }

     /**
```
```
Estimated Gas saved: 2100 gas units.
```




## [G-09]  Non efficient zero initialization
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

### Proof of concept
Using a simple Remix test

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;


contract InitializeDefaultValue {
    
    uint256 num = 0;

}
```
```
Deployment gas cost: 14669
```


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract NoInitializeDefaultValue {
    
    uint256 num;

}
```
```
Deployment gas cost: 12464
```

### Instances
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L67
```solidity
file: revolution/src/MaxHeap.sol

67:    uint256 public size = 0;
```

```diff
diff --git a/packages/revolution/src/MaxHeap.sol b/packages/revolution/src/MaxHeap.sol
index 5dfb81c..5fb9125 100644
--- a/packages/revolution/src/MaxHeap.sol
+++ b/packages/revolution/src/MaxHeap.sol
@@ -64,7 +64,7 @@ contract MaxHeap is VersionedContract, UUPS, Ownable2StepUpgradeable, Reentrancy
     /// @notice Struct to represent an item in the heap by it's ID
     mapping(uint256 => uint256) public heap;

-    uint256 public size = 0;
+    uint256 public size;

     /// @notice Mapping to keep track of the value of an item in the heap
     mapping(uint256 => uint256) public valueMapping;
```




## [G-10] Expression `` is cheaper than new bytes(0)

### 2 Instances
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L191
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L196
```solidity
file: revolution/src/ERC20TokenEmitter.sol

152:    function buyToken(
153:        address[] calldata addresses,
154:        uint[] calldata basisPointSplits,
155:        ProtocolRewardAddresses calldata protocolRewardsRecipients
156:    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
.
.
.
190:        //Deposit funds to treasury
191:        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0)); //@audit `` is cheaper than new bytes(0)
192:        require(success, "Transfer failed.");
193:
194:        //Transfer ETH to creators
195:        if (creatorDirectPayment > 0) {
196:            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0)); //@audit `` is cheaper than new bytes(0)
197:            require(success, "Transfer failed.");
198:        }
.
.
.
230:    }
```

```diff
diff --git a/packages/revolution/src/ERC20TokenEmitter.sol b/packages/revolution/src/ERC20TokenEmitter.sol
index e6f7d46..958cbc2 100644
--- a/packages/revolution/src/ERC20TokenEmitter.sol
+++ b/packages/revolution/src/ERC20TokenEmitter.sol
@@ -188,12 +188,12 @@ contract ERC20TokenEmitter is
         if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

         //Deposit funds to treasury
-        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
+        (bool success, ) = treasury.call{ value: toPayTreasury }("");
         require(success, "Transfer failed.");

         //Transfer ETH to creators
         if (creatorDirectPayment > 0) {
-            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
+            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }("");
             require(success, "Transfer failed.");
         }
```







## [G-11]  Move lesser gas costing require checks to the top Require() or revert() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

#### Please note these instances were not incluede in the bots reports

### 4 Instances
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L308-#L309

The require statements of the `CultureIndex._vote()` function should be re-orderd the statement `require(voter != address(0), "Invalid voter address")` should move to the top of the top function as its the less gas consuming statement of the bunch. So that in scenarios in which it fails the EVM would waste gas executing `require(pieceId < _currentPieceId, "Invalid piece ID")` which involves an `SLOAD`. The diff below shows how the code should be refactored:

```solidity
file: revolution/src/CultureIndex.sol

307:    function _vote(uint256 pieceId, address voter) internal {
308:        require(pieceId < _currentPieceId, "Invalid piece ID");
309:        require(voter != address(0), "Invalid voter address");  //@audit cheaper require statement should move to function top
310:        require(!pieces[pieceId].isDropped, "Piece has already been dropped");
311:        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
312:
313:        uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);
314:        require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");
315:
316:        votes[pieceId][voter] = Vote(voter, weight);
317:        totalVoteWeights[pieceId] += weight;
318:
319:        uint256 totalWeight = totalVoteWeights[pieceId];
320:
321:        // TODO add security consideration here based on block created to prevent flash attacks on drops?
322:        maxHeap.updateValue(pieceId, totalWeight);
323:        emit VoteCast(pieceId, voter, weight, totalWeight);
324:    }
```

```diff
diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
index 88f3338..beb7bc3 100644
--- a/packages/revolution/src/CultureIndex.sol
+++ b/packages/revolution/src/CultureIndex.sol
@@ -305,8 +305,8 @@ contract CultureIndex is
      * Emits a VoteCast event upon successful execution.
      */
     function _vote(uint256 pieceId, address voter) internal {
-        require(pieceId < _currentPieceId, "Invalid piece ID");
         require(voter != address(0), "Invalid voter address");
+        require(pieceId < _currentPieceId, "Invalid piece ID");
         require(!pieces[pieceId].isDropped, "Piece has already been dropped");
         require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
```
```
Estimated Gas saved: 2100 gas units
```


- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L158-#L162

The require statements of the `ERC20TokenEmitter.ERC20TokenEmitter()` function should be re-orderd the statement `require(msg.value > 0, "Must send ether")` should move to the top of the top function as its the less gas consuming statement of the bunch. So that in scenarios in which it fails the EVM would waste gas executing `require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens")` which involves 2 `SLOAD`. The diff below shows how the code should be refactored:

```solidity
file: revolution/src/ERC20TokenEmitter.sol

152:    function buyToken(
153:        address[] calldata addresses,
154:        uint[] calldata basisPointSplits,
155:        ProtocolRewardAddresses calldata protocolRewardsRecipients
156:    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
157:        //prevent treasury from paying itself
158:        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");
159:
160:        require(msg.value > 0, "Must send ether");  //@audit should move to function top
161:        // ensure the same number of addresses and bps
162:        require(addresses.length == basisPointSplits.length, "Parallel arrays required");
.
.
.
230:    }
```

```diff
diff --git a/packages/revolution/src/ERC20TokenEmitter.sol b/packages/revolution/src/ERC20TokenEmitter.sol
index e6f7d46..8d777bb 100644
--- a/packages/revolution/src/ERC20TokenEmitter.sol
+++ b/packages/revolution/src/ERC20TokenEmitter.sol
@@ -154,12 +154,14 @@ contract ERC20TokenEmitter is
         uint[] calldata basisPointSplits,
         ProtocolRewardAddresses calldata protocolRewardsRecipients
     ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
+
+        require(msg.value > 0, "Must send ether");
+        require(addresses.length == basisPointSplits.length, "Parallel arrays required");
         //prevent treasury from paying itself
         require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

-        require(msg.value > 0, "Must send ether");
         // ensure the same number of addresses and bps
-        require(addresses.length == basisPointSplits.length, "Parallel arrays required");
+

         // Get value left after protocol rewards
         uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
```


- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L218-#L222

The require statements of the `AuctionHouse.setCreatorRateBps()` function should be re-orderd the statement `require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000")` should move to the top of the top function as its the less gas consuming statement of the bunch. So that in scenarios in which it fails the EVM would waste gas executing `require(_creatorRateBps >= minCreatorRateBps, "Creator rate must be greater than or equal to minCreatorRateBps")` which involves an `SLOAD`. The diff below shows how the code should be refactored:

```solidity
file: revolution/src/AuctionHouse.sol

217:    function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
218:        require(
219:            _creatorRateBps >= minCreatorRateBps,
220:            "Creator rate must be greater than or equal to minCreatorRateBps"
221:        );
222:        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
223:        creatorRateBps = _creatorRateBps;
224:
225:        emit CreatorRateBpsUpdated(_creatorRateBps);
226:        }    
```

```diff
diff --git a/packages/revolution/src/AuctionHouse.sol b/packages/revolution/src/AuctionHouse.sol
index 55e55a1..b5f03ce 100644
--- a/packages/revolution/src/AuctionHouse.sol
+++ b/packages/revolution/src/AuctionHouse.sol
@@ -215,11 +215,12 @@ contract AuctionHouse is
      * @param _creatorRateBps New creator rate in basis points.
      */
     function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
+        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
         require(
             _creatorRateBps >= minCreatorRateBps,
             "Creator rate must be greater than or equal to minCreatorRateBps"
         );
-        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
+
         creatorRateBps = _creatorRateBps;

         emit CreatorRateBpsUpdated(_creatorRateBps);
```
```
Estimated gas saved: 2100 gas units
```

- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L234-#L235

The require statements of the `AuctionHouse.setMinCreatorRateBps()` function should be re-orderd the statement `require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000")` should move to the top of the top function as its the less gas consuming statement of the bunch. So that in scenarios in which it fails the EVM would waste gas executing `require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate")` which involves an `SLOAD`. The diff below shows how the code should be refactored:

```solidity
file: revolution/src/AuctionHouse.sol

233:    function setMinCreatorRateBps(uint256 _minCreatorRateBps) external onlyOwner {
234:        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
235:        require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");
236:
237:        //ensure new min rate cannot be lower than previous min rate
238:        require(
239:            _minCreatorRateBps > minCreatorRateBps,
240:            "Min creator rate must be greater than previous minCreatorRateBps"
241:        );
242:
243:        minCreatorRateBps = _minCreatorRateBps;
244:
245:        emit MinCreatorRateBpsUpdated(_minCreatorRateBps);
246:    }
```

```diff
diff --git a/packages/revolution/src/AuctionHouse.sol b/packages/revolution/src/AuctionHouse.sol
index 55e55a1..2e0579e 100644
--- a/packages/revolution/src/AuctionHouse.sol
+++ b/packages/revolution/src/AuctionHouse.sol
@@ -231,8 +231,9 @@ contract AuctionHouse is
      * @param _minCreatorRateBps New minimum creator rate in basis points.
      */
     function setMinCreatorRateBps(uint256 _minCreatorRateBps) external onlyOwner {
-        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
         require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");
+        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
+

         //ensure new min rate cannot be lower than previous min rate
         require(
```




## [G-12] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require or if statement
The solidity compiler introduces some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked blocks to have some gas savings.
#### Please note these instances were not found in the bots reports


### Instances
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L79-#L80
```solidity
file: revolution/src/MaxHeap.sol

78:    function parent(uint256 pos) private pure returns (uint256) {
79:        require(pos != 0, "Position should not be zero");
80:        return (pos - 1) / 2;    //@audit can be unchecked since (pos-1) cannot underflow
81:    }
```

- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L160
```solidity
file: 

156:    function extractMax() external onlyAdmin returns (uint256, uint256) {
157:        require(size > 0, "Heap is empty");
158:
159:        uint256 popped = heap[0];
160:        heap[0] = heap[--size];    //@audit can be unchecked since (--size) cannot underflow
161:        maxHeapify(0);
162:
163:        return (popped, valueMapping[popped]);
164:    }
```



## [G-13] Avoid zero to non-zero storage writes
When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.

### 5 Instances

1. #### Refactor `MaxHeap` contract to avoid zero to non-zero storage writes
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L16

When declaring the variable `admin` the default value is applied since we do not assign anything, the default value of address is address(0). We then call the function initialize() and here we assign the value of `_admin` to `admin`. This means our value is updated from `address(0)`  to  the value of `_admin`. Since the initialize() function will always be called first, we can initialize our variable `admin`  with `address(1)` during declaration which should avoid the zero to non zero writes. The diff below shows how the code could be refactored: 

```solidity
file: revolution/src/MaxHeap.sol

16:    address public admin;
.
.
.
55:    function initialize(address _initialOwner, address _admin) public initializer {
56:        require(msg.sender == address(manager), "Only manager can initialize");
57:
58:        admin = _admin;
59:
60:        __Ownable_init(_initialOwner);
61:        __ReentrancyGuard_init();
62:  }
    
```

```diff
diff --git a/packages/revolution/src/MaxHeap.sol b/packages/revolution/src/MaxHeap.sol
index 5dfb81c..e4a20a8 100644
--- a/packages/revolution/src/MaxHeap.sol
+++ b/packages/revolution/src/MaxHeap.sol
@@ -13,7 +13,7 @@ import { VersionedContract } from "./version/VersionedContract.sol";
 /// @author Written by rocketman and gpt4
 contract MaxHeap is VersionedContract, UUPS, Ownable2StepUpgradeable, ReentrancyGuardUpgradeable {
     /// @notice The parent contract that is allowed to update the data store
-    address public admin;
+    address public admin = address(1);
```

2. #### Refactor `CultureIndex` contract to avoid zero to non-zero storage writes
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L36
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L39
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L42
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L45
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L51
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L54
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L57
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L60
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L78


3. #### Refactor `ERC20TokenEmitter` contract to avoid zero to non-zero storage writes
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L25
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L28
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L31
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L34
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L48


4. #### Refactor `AuctionHouse` contract to avoid zero to non-zero storage writes
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L48
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L51
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L54
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L56
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L60
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L63
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L66
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L69
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L72
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L75


5. #### Refactor `AuctionHouse` contract to avoid zero to non-zero storage writes
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L42
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L45
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L48
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol



## [G-14] Refactor Functions to avoid unnecessary copy of storage struct to memory.

In the following functions a complete struct is being copied to memory to end up using only some of its attributes.

### 1 Instance

1. #### Refactor `AuctionHouse.createBid()` to avoid unnecessary copy of storage struct to memory
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L171-#L200

In the `AuctionHouse.createBid()` function as shown below the storage struct `auction` was copied into memory variable `_auction` but only the `verbId`, `endtime`, `amount`, `bidder` attributes were used in the function thereby making this process gas expensive as some attributes (`starttime`, `settled`) were copied into memory but were not used in the function. We can refactor the `AuctionHouse.createBid()` function to be more gas efficient by not copying the `auction` storage struct into memory rather we cache only the `auction` attributes needed in the function. The diff below shows how the code could be refactored:

```solidity
file: revolution/src/AuctionHouse.sol

171:    function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
172:        IAuctionHouse.Auction memory _auction = auction;
173:
174:        //require bidder is valid address
175:        require(bidder != address(0), "Bidder cannot be zero address");
176:        require(_auction.verbId == verbId, "Verb not up for auction");
177:        //slither-disable-next-line timestamp
178:        require(block.timestamp < _auction.endTime, "Auction expired");
179:        require(msg.value >= reservePrice, "Must send at least reservePrice");
180:        require(
181:            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
182:            "Must send more than last bid by minBidIncrementPercentage amount"
183:        );
184:
185:        address payable lastBidder = _auction.bidder;
186:
187:        auction.amount = msg.value;
188:        auction.bidder = payable(bidder);
189:
190:        // Extend the auction if the bid was received within `timeBuffer` of the auction end time
191:        bool extended = _auction.endTime - block.timestamp < timeBuffer;
192:        if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
193:
194:        // Refund the last bidder, if applicable
195:        if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);
196:
197:        emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);
198:
199:        if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);
200:    }
```

```diff
diff --git a/packages/revolution/src/AuctionHouse.sol b/packages/revolution/src/AuctionHouse.sol
index 55e55a1..76818ae 100644
--- a/packages/revolution/src/AuctionHouse.sol
+++ b/packages/revolution/src/AuctionHouse.sol
@@ -169,34 +169,36 @@ contract AuctionHouse is
      * @param bidder The address of the bidder.
      */
     function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
-        IAuctionHouse.Auction memory _auction = auction;
+
+        uint256 _auctionVerbId = auction.verbId;
+        uint256 _auctionEndtime = auction.endTime;
+        uint256 _auctionAmount = auction.amount;
+        address payable _auctionbidder = auction.bidder;

         //require bidder is valid address
         require(bidder != address(0), "Bidder cannot be zero address");
-        require(_auction.verbId == verbId, "Verb not up for auction");
+        require(_auctionVerbId == verbId, "Verb not up for auction");
         //slither-disable-next-line timestamp
-        require(block.timestamp < _auction.endTime, "Auction expired");
+        require(block.timestamp < _auctionEndtime, "Auction expired");
         require(msg.value >= reservePrice, "Must send at least reservePrice");
         require(
-            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
+            msg.value >= _auctionAmount + ((_auctionAmount * minBidIncrementPercentage) / 100),
             "Must send more than last bid by minBidIncrementPercentage amount"
         );

-        address payable lastBidder = _auction.bidder;
-
         auction.amount = msg.value;
         auction.bidder = payable(bidder);

         // Extend the auction if the bid was received within `timeBuffer` of the auction end time
-        bool extended = _auction.endTime - block.timestamp < timeBuffer;
-        if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
+        bool extended = _auctionEndtime - block.timestamp < timeBuffer;
+        if (extended) auction.endTime = _auctionEndtime = block.timestamp + timeBuffer;

         // Refund the last bidder, if applicable
-        if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);
+        if (_auctionbidder != address(0)) _safeTransferETHWithFallback(_auctionbidder, _auctionAmount);
-        emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);
+        emit AuctionBid(_auctionVerbId, bidder, msg.sender, msg.value, extended);

-        if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);
+        if (extended) emit AuctionExtended(_auctionVerbId, _auctionEndtime);
     }
```



## [G-15] Refactor external/internal function to avoid unnecessary SLOAD
The functions below read storage slots that are previously read in the functions that invoke them. We can refactor the external/internal functions to pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.

### Instances
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L86-#L151




## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.