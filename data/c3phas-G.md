# Codebase optimization report

## Table of Contents

- [Codebase optimization report](#codebase-optimization-report)
  - [Table of Contents](#table-of-contents)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Pack variables(save 2 Slots: 4200 Gas)](#pack-variablessave-2-slots-4200-gas)
  - [We can save 3 SLOTS(6300 Gas)](#we-can-save-3-slots6300-gas)
  - [Pack `quorumVotesBPS` with `dropperAdmin` by reducing size of `quorumVotesBPS`(Save 1 SLOT: 2100 Gas)](#pack-quorumvotesbps-with-dropperadmin-by-reducing-size-of-quorumvotesbpssave-1-slot-2100-gas)
  - [The following finding touches an OOS file(interface) but is greatly used in very sensitive functions(feel free to exclude it when grading)](#the-following-finding-touches-an-oos-fileinterface-but-is-greatly-used-in-very-sensitive-functionsfeel-free-to-exclude-it-when-grading)
    - [We can pack the struct here by reducing the size of timestamps(save 2 SLOTS: 4200 Gas)](#we-can-pack-the-struct-here-by-reducing-the-size-of-timestampssave-2-slots-4200-gas)
  - [Avoid making two external calls here](#avoid-making-two-external-calls-here)
  - [Cache state variables in memory(not found by bots)](#cache-state-variables-in-memorynot-found-by-bots)
    - [We can cache `cultureIndex` to save 2 Sloads(~200 Gas)](#we-can-cache-cultureindex-to-save-2-sloads200-gas)
    - [We can cache maxheap to save 1 sload(~100 Gas)](#we-can-cache-maxheap-to-save-1-sload100-gas)
    - [Cache `creatorsAddress` and `treasury`(save 4 Sloads: ~400 Gas)](#cache-creatorsaddress-and-treasurysave-4-sloads-400-gas)
  - [we can cache `verbs` to save 1 Sload(~100 Gas)](#we-can-cache-verbs-to-save-1-sload100-gas)
  - [Redundant check should be removed](#redundant-check-should-be-removed)
  - [Return the local variable(Avoid reading from storage: Save 1 SLOAD)](#return-the-local-variableavoid-reading-from-storage-save-1-sload)
  - [Optimizing check order for cost efficient function execution](#optimizing-check-order-for-cost-efficient-function-execution)
    - [Reorder the checks to validate `msg.value` first before reading state variables](#reorder-the-checks-to-validate-msgvalue-first-before-reading-state-variables)
    - [Revert cheaply by validating function parameters first](#revert-cheaply-by-validating-function-parameters-first)
    - [Avoid reading any state variables before validating all parameters first](#avoid-reading-any-state-variables-before-validating-all-parameters-first)
    - [Validate function parameters first](#validate-function-parameters-first)
  - [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)


## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

**We have made sure that we do not repeat issues reported by the bot.**

## Pack variables(save 2 Slots: 4200 Gas)

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L42-L48
```solidity
File: /packages/revolution/src/ERC20TokenEmitter.sol
42:    uint256 public creatorRateBps;

45:    uint256 public entropyRateBps;

48:    address public creatorsAddress;
```
we can reduce the size of `creatorRateBps` and `entropyRateBps` to `uint32` since their values are constrained to be less than `10_000` 
We can see this enforced in the following 
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L288-L292
```solidity
288:    function setEntropyRateBps(uint256 _entropyRateBps) external onlyOwner {
289:        require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

291:        emit EntropyRateBpsUpdated(entropyRateBps = _entropyRateBps);
292:    }
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L299-L303
```solidity
299:    function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
300:        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");

302:        emit CreatorRateBpsUpdated(creatorRateBps = _creatorRateBps);
303:    }
```

We can refactor and pack as follows
```diff
     // The split of the purchase that is reserved for the creator of the Verb in basis points
-    uint256 public creatorRateBps;
+    uint32 public creatorRateBps; 

     // The split of (purchase proceeds * creatorRate) that is sent to the creator as ether in basis points
-    uint256 public entropyRateBps;
+    uint32 public entropyRateBps;

     // The account or contract to pay the creator reward to
     address public creatorsAddress;
```


**Similar instance**

## We can save 3 SLOTS(6300 Gas)

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L48-L72
```solidity
File: /packages/revolution/src/AuctionHouse.sol
48:    IVerbsToken public verbs;

51:    IERC20TokenEmitter public erc20TokenEmitter;

66:    uint256 public creatorRateBps;

69:    uint256 public minCreatorRateBps;

72:    uint256 public entropyRateBps;
```
similar to the previous instance, when setting `creatorrateBps,minCreatorRateBps,entropyRateBps` we constrain their value to be less or equal to `10_000` therefore `uint32` should be more than enough.
We can see the constrains being enforced in the following setters

**`creatorRateBps` setter**
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L217-L226
```solidity
    function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
    
        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
        creatorRateBps = _creatorRateBps;
```

**`minCreatorRateBps` setter**
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L233-L246
```solidity
    function setMinCreatorRateBps(uint256 _minCreatorRateBps) external onlyOwner {
        require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");

        minCreatorRateBps = _minCreatorRateBps;
```

**`entropyRateBps` setter**
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L253-L258
```solidity
    function setEntropyRateBps(uint256 _entropyRateBps) external onlyOwner {
        require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

        entropyRateBps = _entropyRateBps;
```

We are choosing to pack with the variable `verbs` since they are being accessed in the same transaction

```diff
-    // The Verbs ERC721 token contract
-    IVerbsToken public verbs;

     // The ERC20 governance token
     IERC20TokenEmitter public erc20TokenEmitter;
@@ -54,25 +53,27 @@ contract AuctionHouse is
     address public WETH;

     // The split of the winning bid that is reserved for the creator of the Verb in basis points
-    uint256 public creatorRateBps;
+    uint32 public creatorRateBps; 

     // The all time minimum split of the winning bid that is reserved for the creator of the Verb in basis points
-    uint256 public minCreatorRateBps;
+    uint32 public minCreatorRateBps;

     // The split of (auction proceeds * creatorRate) that is sent to the creator as ether in basis points
-    uint256 public entropyRateBps;
+    uint32 public entropyRateBps;
+        // The Verbs ERC721 token contract
+    IVerbsToken public verbs;
```

This way we can save 3 SLOTS: note more packing was reported by the bot


## Pack `quorumVotesBPS` with `dropperAdmin` by reducing size of `quorumVotesBPS`(Save 1 SLOT: 2100 Gas)

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L53-L78
```solidity
File: /packages/revolution/src/CultureIndex.sol
53:    uint256 public quorumVotesBPS;

57:    string public name;

78:    address public dropperAdmin;
```

The variable `quorumVotesBPS` can fit inside a lower type eg `uint64` as their value is constrained to `6_000`  see the following
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L119-L137
```solidity
119:        require(_cultureIndexParams.quorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "invalid quorum bps");

137:        quorumVotesBPS = _cultureIndexParams.quorumVotesBPS;
```
Note, we have a check that ensures the value assigned to `quorumVotesBPS` is less than `MAX_QUORUM_VOTES_BPS`.
`MAX_QUORUM_VOTES_BPS` is a constant variable defined as follows
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L48
```solidity
48:    uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%
```

We also have a setter for the variable `quorumVotesBPS` which is implemented as follows
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L498-L503
```solidity
498:    function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external onlyOwner {
499:        require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");
500:        emit QuorumVotesBPSSet(quorumVotesBPS, newQuorumVotesBPS);

502:        quorumVotesBPS = newQuorumVotesBPS;
503:    }
```
Again, we see the check that constrains the value to be less than `6000`

We are suggesting packing with `addess dropperAdmin` as they are accessed together. Refactor the code as follows
```diff
@@ -51,7 +51,9 @@ contract CultureIndex is
     uint256 public minVoteWeight;

     /// @notice The basis point number of votes in support of a art piece required in order for a quorum to be reached and for an art piece to be dropped.
-    uint256 public quorumVotesBPS;
+    uint64 public quorumVotesBPS;//@audit reduce me and pack me with address dropperAdmin
+        // The address that is allowed to drop art pieces
+    address public dropperAdmin;

     /// @notice The name of the culture index
     string public name;
@@ -74,8 +76,7 @@ contract CultureIndex is
     // Constant for max number of creators
     uint256 public constant MAX_NUM_CREATORS = 100;

-    // The address that is allowed to drop art pieces
-    address public dropperAdmin;
```

## The following finding touches an OOS file(interface) but is greatly used in very sensitive functions(feel free to exclude it when grading)

### We can pack the struct here by reducing the size of timestamps(save 2 SLOTS: 4200 Gas)

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L78
```solidity
File: /packages/revolution/src/AuctionHouse.sol
78:    IAuctionHouse.Auction public auction;
```
The above variable defines a struct variable that is declared in an OOS file. Due to the huge impact it might have we choose to include it in the report. This affects mostly inscope functions such as `createBid(), _createAuction(),_settleAuction()` which are vital 
The struct auction is defined as follows

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/IAuctionHouse.sol#L23-L36
```solidity
File: /packages/revolution/src/interfaces/IAuctionHouse.sol
23:    struct Auction {
24:        // ID for the Verb (ERC721 token ID)
25:        uint256 verbId;
26:        // The current highest bid amount
27:        uint256 amount;
28:        // The time that the auction started
29:        uint256 startTime;
30:        // The time that the auction is scheduled to end
31:        uint256 endTime;
32:        // The address of the current highest bid
33:        address payable bidder;
34:        // Whether or not the auction has been settled
35:        bool settled;
36:    }
```


We can reduce the size of `startTime` and `endtime` to `uint40` and pack them with `bidder` and `settled` 
Since these are timestamps this should be fine and would save us 2 SLOTS: 4200 Gas

If we are feeling a bit generous, we can choose `uint48`  or `uint64` saving us 1 SLOT


```diff
diff --git a/packages/revolution/src/interfaces/IAuctionHouse.sol b/packages/revolution/src/interfaces/IAuctionHouse.sol
index a8378e8..5f05dc7 100644
--- a/packages/revolution/src/interfaces/IAuctionHouse.sol
+++ b/packages/revolution/src/interfaces/IAuctionHouse.sol
@@ -26,9 +26,9 @@ interface IAuctionHouse {
         // The current highest bid amount
         uint256 amount;
         // The time that the auction started
-        uint256 startTime;
+        uint40 startTime;
         // The time that the auction is scheduled to end
-        uint256 endTime;
+        uint40 endTime;
         // The address of the current highest bid
         address payable bidder;
```

## Avoid making two external calls here

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L370C22-L371
```solidity
File: /packages/revolution/src/AuctionHouse.sol
370:                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
371:                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
```
We can refactor the code to just make 1 call then use it to get the values we are interested in
```diff
-                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
-                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

+
+                ICultureIndex.ArtPiece memory _verbs = verbs.getArtPieceById(_auction.verbId);
+                uint256 numCreators = _verbs.creators.length;
+                address deployer = _verbs.sponsor;
```


## Cache state variables in memory(not found by bots)

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L281-L292

### We can cache `cultureIndex` to save 2 Sloads(~200 Gas)

```solidity
File: /packages/revolution/src/VerbsToken.sol
281:    function _mintTo(address to) internal returns (uint256) {
282:        ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

286:        require(
287:            artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
288:            "Creator array must not be > MAX_NUM_CREATORS"
289:        );

291:        // Use try/catch to handle potential failure
292:        try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
```
We can cache the variable `cultureIndex` instead of reading it from storage 3 times


```diff
diff --git a/packages/revolution/src/VerbsToken.sol b/packages/revolution/src/VerbsToken.sol
index 7bd9527..3915607 100644
--- a/packages/revolution/src/VerbsToken.sol
+++ b/packages/revolution/src/VerbsToken.sol
@@ -279,17 +279,18 @@ contract VerbsToken is
      * @notice Mint a Verb with `verbId` to the provided `to` address. Pulls the top voted art piece from the CultureIndex.
      */
     function _mintTo(address to) internal returns (uint256) {
-        ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();
+         ICultureIndex  _cultureIndex = cultureIndex;
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


https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L486-L491

### We can cache maxheap to save 1 sload(~100 Gas)

```solidity
File: /packages/revolution/src/CultureIndex.sol
486:    function topVotedPieceId() public view returns (uint256) {
487:        require(maxHeap.size() > 0, "Culture index is empty");
488:        //slither-disable-next-line unused-return
489:        (uint256 pieceId, ) = maxHeap.getMax();
490:        return pieceId;
491:    }
```

```diff
     function topVotedPieceId() public view returns (uint256) {
-        require(maxHeap.size() > 0, "Culture index is empty");
+        MaxHeap _maxHeap = maxHeap;
+        require(_maxHeap.size() > 0, "Culture index is empty");
         //slither-disable-next-line unused-return
-        (uint256 pieceId, ) = maxHeap.getMax();
+        (uint256 pieceId, ) = _maxHeap.getMax();//@audit cache maxheap
         return pieceId;
     }
```


https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L158-L203

### Cache `creatorsAddress` and `treasury`(save 4 Sloads: ~400 Gas)

```solidity
File: /packages/revolution/src/ERC20TokenEmitter.sol
158:        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");


187:        emittedTokenWad += totalTokensForBuyers;
188:        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

191:        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));


195:        if (creatorDirectPayment > 0) {
196:            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
197:            require(success, "Transfer failed.");
198:        }

201:        if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
202:            _mint(creatorsAddress, uint256(totalTokensForCreators));
203:        }
```

`creatorsAddress` is being read 4 times, so 4 SLOADS(400 Gas), `treasury` is being read 2 times(200 Gas)


https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L356-L371

## we can cache `verbs` to save 1 Sload(~100 Gas)

```solidity
File: /packages/revolution/src/AuctionHouse.sol
356:        } else {
357:            //If no one has bid, burn the Verb
358:            if (_auction.bidder == address(0))
359:                verbs.burn(_auction.verbId);
360:                //If someone has bid, transfer the Verb to the winning bidder
361:            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);


363:            if (_auction.amount > 0) {

368:                uint256 creatorsShare = _auction.amount - auctioneerPayment;

370:                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
371:                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
```

The number of sloads to be saved here depends on the path the code takes so we can focus on the second bit where have 2 sloads happening

```diff
             if (_auction.amount > 0) {
+                IVerbsToken  _verbs = verbs;
                 // Ether going to owner of the auction
                 uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;

                 //Total amount of ether going to creator
-                uint256 creatorsShare = _auction.amount - auctioneerPayment;
+                uint256 creatorsShare = _auction.amount - auctioneerPayment;//@audit should never overflow

-                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
-                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
+                uint256 numCreators = _verbs.getArtPieceById(_auction.verbId).creators.length;
+                address deployer = _verbs.getArtPieceById(_auction.verbId).sponsor;

```

## Redundant check should be removed

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L78-L81
```solidity
File: /packages/revolution/src/MaxHeap.sol
78:    function parent(uint256 pos) private pure returns (uint256) {
79:        require(pos != 0, "Position should not be zero");
80:        return (pos - 1) / 2;
81:    }
```

The first require statement is not necessary. Whenever this function is called, there is no possibility of having `pos` being `0` which makes this check redundant

We call this function in the following parts 
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L124-L128
```solidity
        uint256 current = size;
        while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
            swap(current, parent(current));
            current = parent(current);
        }
```
Note, `current` which is the value we pass to our `parent` function is being checked against `0` at the beginning of the loop. The only way we can execute the `parent()` function is when `current` `>` `0`

The other instance is https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L146-L149
```solidity
            while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
                swap(position, parent(position));
                position = parent(position);
            }
```
we also ensure `position != 0` which is the value we pass to our `parent()` function

## Return the local variable(Avoid reading from storage: Save 1 SLOAD)

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L209-L248
```solidity
File: /packages/revolution/src/CultureIndex.sol
209:    function createPiece(
210:        ArtPieceMetadata calldata metadata,
211:        CreatorBps[] calldata creatorArray
212:    ) public returns (uint256) {

218:        uint256 pieceId = _currentPieceId++;

223:        ArtPiece storage newPiece = pieces[pieceId];

225:        newPiece.pieceId = pieceId;

236:        for (uint i; i < creatorArrayLength; i++) {
237:            newPiece.creators.push(creatorArray[i]);
238:        }

247:        return newPiece.pieceId;
248:    }
```

On Line 225, we set the value of `newPiece.pieceId` (a state variable) to be equal to `pieceId`(a local variable). It is cheaper to read local variables as compared to state variables. We can therefore save 1 SLOAD by refactoring the code as follows.
```diff
@@ -215,7 +215,7 @@ contract CultureIndex is
-        return newPiece.pieceId;
+        return pieceId;
     }
```


## Optimizing check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L152-L162

### Reorder the checks to validate `msg.value` first before reading state variables

```solidity
File: /packages/revolution/src/ERC20TokenEmitter.sol
152:    function buyToken(
153:        address[] calldata addresses,
154:        uint[] calldata basisPointSplits,
155:        ProtocolRewardAddresses calldata protocolRewardsRecipients
156:    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
157:        //prevent treasury from paying itself
158:        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

160:        require(msg.value > 0, "Must send ether");
161:        // ensure the same number of addresses and bps
162:        require(addresses.length == basisPointSplits.length, "Parallel arrays required");
```


```diff
@@ -154,12 +154,11 @@ contract ERC20TokenEmitter is
         uint[] calldata basisPointSplits,
         ProtocolRewardAddresses calldata protocolRewardsRecipients
     ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
-        //prevent treasury from paying itself
-        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");
-
-        require(msg.value > 0, "Must send ether");
+        require(msg.value > 0, "Must send ether");
         // ensure the same number of addresses and bps
         require(addresses.length == basisPointSplits.length, "Parallel arrays required");
+        //prevent treasury from paying itself
+        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

         // Get value left after protocol rewards
         uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
```


https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L171-L175

### Revert cheaply by validating function parameters first

```solidity
File: /packages/revolution/src/AuctionHouse.sol
171:    function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
172:        IAuctionHouse.Auction memory _auction = auction;

174:        //require bidder is valid address
175:        require(bidder != address(0), "Bidder cannot be zero address");
```

Avoid reading any state variables if we can revert cheaply on function parameter checks

```diff
     function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
+        //require bidder is valid address
+        require(bidder != address(0), "Bidder cannot be zero address");//@audit
 move up
         IAuctionHouse.Auction memory _auction = auction;

-        //require bidder is valid address
-        require(bidder != address(0), "Bidder cannot be zero address");
         require(_auction.verbId == verbId, "Verb not up for auction");
```


https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L218-L222

### Avoid reading any state variables before validating all parameters first

```solidity
File: /packages/revolution/src/AuctionHouse.sol
218:        require(
219:            _creatorRateBps >= minCreatorRateBps,
220:            "Creator rate must be greater than or equal to minCreatorRateBps"
221:        );
222:        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
```


```diff
     function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
+        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
         require(
             _creatorRateBps >= minCreatorRateBps,
             "Creator rate must be greater than or equal to minCreatorRateBps"
         );
-        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
         creatorRateBps = _creatorRateBps;
```


https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L234-L235

### Validate function parameters first

```solidity
File: /packages/revolution/src/AuctionHouse.sol
234:        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
235:        require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");
```
We should validate function parameters first before reading any state variables
```diff
     function setMinCreatorRateBps(uint256 _minCreatorRateBps) external onlyOwner {
-        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
         require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");
+        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");

```

## Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L399-L409
```solidity
File:/packages/revolution/src/AuctionHouse.sol
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
```

We can wrap the whole thing with unchecked blocks since the only operation we need to be concerned with `creatorsShare - ethPaidToCreators ` cannot underflow since we have a check for `creatorsShare > ethPaidToCreators`

