**Note:** I've removed the specific instances from the bot race and 4naly3er

## Summary

### Gas Optimizations

| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [[G&#x2011;01](#g01--has-the-same-value-as-new-bytes0-but-costs-less-gas)] | `""` has the same value as `new bytes(0)` but costs less gas | 2 |  518 |
| [[G&#x2011;02](#g02-assembly-use-scratch-space-for-building-calldata)] | Assembly: Use scratch space for building calldata | 14 |  3080 |
| [[G&#x2011;03](#g03-avoid-transferring-amounts-of-zero-in-order-to-save-gas)] | Avoid transferring amounts of zero in order to save gas | 1 |  100 |
| [[G&#x2011;04](#g04-combine-mappings-referenced-in-the-same-function-by-the-same-key)] | Combine `mapping`s referenced in the same function by the same key | 4 |  168 |
| [[G&#x2011;05](#g05-assembly-use-scratch-space-when-building-emitted-events-with-two-data-arguments)] | Assembly: Use scratch space when building emitted events with two data arguments | 4 |  60 |
| [[G&#x2011;06](#g06--costs-less-gas-than-)] | `>=` costs less gas than `>` | 9 |  27 |
| [[G&#x2011;07](#g07-stack-variable-is-only-used-once)] | Stack variable is only used once | 5 |  15 |
| [[G&#x2011;08](#g08-requirerevert-strings-longer-than-32-bytes-cost-extra-gas)] | `require()`/`revert()` strings longer than 32 bytes cost extra gas | 1 |  3 |
| [[G&#x2011;09](#g09-consider-using-soladys-token-contracts-to-save-gas)] | Consider using `solady`'s token contracts to save gas | 1 |  - |
| [[G&#x2011;10](#g10-consider-using-soladys-fixedpointmathlib)] | Consider using solady's `FixedPointMathLib` | 18 |  - |
| [[G&#x2011;11](#g11-duplicated-requirerevert-checks-should-be-refactored-to-a-modifier-or-function-to-save-deployment-gas)] | Duplicated `require()`/`revert()` checks should be refactored to a modifier or function to save deployment gas | 20 |  - |
| [[G&#x2011;12](#g12-events-should-be-emitted-outside-of-loops)] | Events should be emitted outside of loops | 1 |  375 |
| [[G&#x2011;13](#g13-multiple-accesses-of-a-memorycalldata-array-should-use-a-local-variable-cache)] | Multiple accesses of a `memory`/`calldata` array should use a local variable cache | 1 |  - |
| [[G&#x2011;14](#g14-reduce-deployment-costs-by-tweaking-contracts-metadata)] | Reduce deployment costs by tweaking contracts' metadata | 7 |  - |
| [[G&#x2011;15](#g15-split-revert-checks-to-save-gas)] | Split `revert` checks to save gas | 2 |  4 |
| [[G&#x2011;16](#g16-use-internal-functions-inside-modifiers-to-save-gas)] | Use internal functions inside modifiers to save gas | 5 |  - |
| [[G&#x2011;17](#g17-using-private-rather-than-public-saves-gas)] | Using `private` rather than `public`, saves gas | 44 |  - |

Total: 139 instances over 17 issues with **4350 gas** saved

Gas totals are estimates based on data from the Ethereum Yellowpaper. The estimates use the lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations


### [G&#x2011;01] `""` has the same value as `new bytes(0)` but costs less gas


*There are 2 instances of this issue:*

```solidity
File: src/ERC20TokenEmitter.sol

191:         (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));

196:             (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));

```
*GitHub*: [191](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L191-L191), [196](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L196-L196)



### [G&#x2011;02] Assembly: Use scratch space for building calldata
If an external call's calldata can fit into two or fewer words, use the scratch space to build the calldata, rather than allowing Solidity to do a memory expansion.

*There are 14 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

355:             verbs.burn(_auction.verbId);

359:                 verbs.burn(_auction.verbId);

370:                 uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;

371:                 address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

385:                         ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];

438:             bool wethSuccess = IWETH(WETH).transfer(_to, _amount);

454:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [355](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L355-L355), [359](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L359-L359), [370](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L370-L370), [371](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L371-L371), [385](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L385-L385), [438](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L438-L438), [454](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L454-L454)

```solidity
File: src/CultureIndex.sol

289:         return _calculateVoteWeight(erc20VotingToken.getVotes(account), erc721VotingToken.getVotes(account));

294              _calculateVoteWeight(
295                  erc20VotingToken.getPastVotes(account, blockNumber),
296                  erc721VotingToken.getPastVotes(account, blockNumber)
297:             );

545:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [289](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L289-L289), [294](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L294-L297), [545](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L545-L545)

```solidity
File: src/ERC20TokenEmitter.sol

109:         token.mint(_to, _amount);

124:         return token.balanceOf(_owner);

```
*GitHub*: [109](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L109-L109), [124](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L124-L124)

```solidity
File: src/MaxHeap.sol

183:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [183](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L183-L183)

```solidity
File: src/VerbsToken.sol

330:         require(manager.isRegisteredUpgrade(_getImplementation(), _newImpl), "Invalid upgrade");

```
*GitHub*: [330](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L330-L330)



### [G&#x2011;03] Avoid transferring amounts of zero in order to save gas
Skipping the external call when nothing will be transferred, will save at least **100 gas**

*There is one instance of this issue:*

```solidity
File: src/AuctionHouse.sol

438:             bool wethSuccess = IWETH(WETH).transfer(_to, _amount);

```
*GitHub*: [438](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L438-L438)



### [G&#x2011;04] Combine `mapping`s referenced in the same function by the same key
Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot (i.e. runtime gas savings). Even if the values can't be packed, if both fields are accessed in the same function (as is the case for these instances), combining them can save **~42 gas per access** due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/639032d73e35d7e968ff58d8784bc445) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*There are 4 instances of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit combine into a `struct`: pieces,totalVoteWeights,votes
307      function _vote(uint256 pieceId, address voter) internal {
308          require(pieceId < _currentPieceId, "Invalid piece ID");
309          require(voter != address(0), "Invalid voter address");
310          require(!pieces[pieceId].isDropped, "Piece has already been dropped");
311          require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
312  
313          uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);
314          require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");
315  
316          votes[pieceId][voter] = Vote(voter, weight);
317          totalVoteWeights[pieceId] += weight;
318  
319          uint256 totalWeight = totalVoteWeights[pieceId];
320  
321          // TODO add security consideration here based on block created to prevent flash attacks on drops?
322          maxHeap.updateValue(pieceId, totalWeight);
323          emit VoteCast(pieceId, voter, weight, totalWeight);
324:     }

/// @audit combine into a `struct`: pieces,totalVoteWeights
519      function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
520          require(msg.sender == dropperAdmin, "Only dropper can drop pieces");
521  
522          ICultureIndex.ArtPiece memory piece = getTopVotedPiece();
523          require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");
524  
525          //set the piece as dropped
526          pieces[piece.pieceId].isDropped = true;
527  
528          //slither-disable-next-line unused-return
529          maxHeap.extractMax();
530  
531          emit PieceDropped(piece.pieceId, msg.sender);
532  
533          return pieces[piece.pieceId];
534:     }

```
*GitHub*: [307](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L307-L324), [519](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L519-L534)

```solidity
File: src/MaxHeap.sol

/// @audit combine into a `struct`: positionMapping,valueMapping
119      function insert(uint256 itemId, uint256 value) public onlyAdmin {
120          heap[size] = itemId;
121          valueMapping[itemId] = value; // Update the value mapping
122          positionMapping[itemId] = size; // Update the position mapping
123  
124          uint256 current = size;
125          while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
126              swap(current, parent(current));
127              current = parent(current);
128          }
129          size++;
130:     }

/// @audit combine into a `struct`: positionMapping,valueMapping
136      function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
137          uint256 position = positionMapping[itemId];
138          uint256 oldValue = valueMapping[itemId];
139  
140          // Update the value in the valueMapping
141          valueMapping[itemId] = newValue;
142  
143          // Decide whether to perform upwards or downwards heapify
144          if (newValue > oldValue) {
145              // Upwards heapify
146              while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
147                  swap(position, parent(position));
148                  position = parent(position);
149              }
150          } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify
151:     }

```
*GitHub*: [119](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L119-L130), [136](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L136-L151)



### [G&#x2011;05] Assembly: Use scratch space when building emitted events with two data arguments
Using the [scratch space](https://gist.github.com/IllIllI000/87c4f03139fa03780fa548b8e4b02b5b) for more than one, but at most two words worth of data (non-indexed arguments) will save gas over needing Solidity's abi memory expansion used for emitting normally.

*There are 4 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

326:             emit AuctionCreated(verbId, startTime, endTime);

```
*GitHub*: [326](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L326-L326)

```solidity
File: src/CultureIndex.sol

141:         emit QuorumVotesBPSSet(quorumVotesBPS, _cultureIndexParams.quorumVotesBPS);

323:         emit VoteCast(pieceId, voter, weight, totalWeight);

500:         emit QuorumVotesBPSSet(quorumVotesBPS, newQuorumVotesBPS);

```
*GitHub*: [141](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L141-L141), [323](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L323-L323), [500](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L500-L500)



### [G&#x2011;06] `>=` costs less gas than `>`
The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, [which saves **3 gas**](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde). If `<` is being used, the condition can be inverted. In cases where a for-loop is being used, one can count down rather than up, in order to use the optimal operator

*There are 9 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

384:                     for (uint256 i = 0; i < numCreators; i++) {

```
*GitHub*: [384](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L384-L384)

```solidity
File: src/CultureIndex.sol

185:         for (uint i; i < creatorArrayLength; i++) {

236:         for (uint i; i < creatorArrayLength; i++) {

243:         for (uint i; i < creatorArrayLength; i++) {

355:         for (uint256 i; i < len; i++) {

403:         for (uint256 i; i < len; i++) {

407:         for (uint256 i; i < len; i++) {

```
*GitHub*: [185](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L185-L185), [236](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L236-L236), [243](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L243-L243), [355](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L355-L355), [403](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L403-L403), [407](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L407-L407)

```solidity
File: src/ERC20TokenEmitter.sol

209:         for (uint256 i = 0; i < addresses.length; i++) {

```
*GitHub*: [209](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L209-L209)

```solidity
File: src/VerbsToken.sol

306:             for (uint i = 0; i < artPiece.creators.length; i++) {

```
*GitHub*: [306](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L306-L306)



### [G&#x2011;07] Stack variable is only used once
If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the **3 gas** the extra stack assignment would spend

*There are 5 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

371:                 address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

438:             bool wethSuccess = IWETH(WETH).transfer(_to, _amount);

```
*GitHub*: [371](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L371-L371), [438](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L438-L438)

```solidity
File: src/CultureIndex.sol

375:         bool success = _verifyVoteSignature(from, pieceIds, deadline, v, r, s);

433:         bytes32 digest = _hashTypedDataV4(voteHash);

489:         (uint256 pieceId, ) = maxHeap.getMax();

```
*GitHub*: [375](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L375-L375), [433](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L433-L433), [489](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L489-L489)



### [G&#x2011;08] `require()`/`revert()` strings longer than 32 bytes cost extra gas
Each extra memory word of bytes past the original 32 [incurs an MSTORE](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#consider-having-short-revert-strings) which costs **3 gas**

*There is one instance of this issue:*

```solidity
File: src/AuctionHouse.sol

340:          require(!_auction.settled, "Auction has already been settled");

```
*GitHub*: [340](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L340)



### [G&#x2011;09] Consider using `solady`'s token contracts to save gas
They're written using heavily-optimized assembly, and will your users a lot of gas

*There is one instance of this issue:*

```solidity
File: src/VerbsToken.sol

/// @audit ERC721Upgradeable
33   contract VerbsToken is
34       IVerbsToken,
35       VersionedContract,
36       UUPS,
37       Ownable2StepUpgradeable,
38       ReentrancyGuardUpgradeable,
39       ERC721CheckpointableUpgradeable
40   {
41:      // An address who has permissions to mint Verbs

```
*GitHub*: [33](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L33-L41)



### [G&#x2011;10] Consider using solady's `FixedPointMathLib`
Saves gas, and works to avoid unnecessary [overflows](https://github.com/Vectorized/solady/blob/6cce088e69d6e46671f2f622318102bd5db77a65/src/utils/FixedPointMathLib.sol#L267).

*There are 18 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

181:             msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),

365:                 uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;

390:                         uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);

```
*GitHub*: [181](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L181-L181), [365](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L365-L365), [390](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L390-L390)

```solidity
File: src/CultureIndex.sol

234:         newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;

511              (quorumVotesBPS * _calculateVoteWeight(erc20VotingToken.totalSupply(), erc721VotingToken.totalSupply())) /
512:             10_000;

```
*GitHub*: [234](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L234-L234), [511](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L511-L512)

```solidity
File: src/ERC20TokenEmitter.sol

173:         uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;

177:         uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;

212:                 _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));

279:                 amount: int(((paymentAmount - computeTotalReward(paymentAmount)) * (10_000 - creatorRateBps)) / 10_000)

```
*GitHub*: [173](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L173-L173), [177](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L177-L177), [212](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L212-L212), [279](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L279-L279)

```solidity
File: src/MaxHeap.sol

80:          return (pos - 1) / 2;

```
*GitHub*: [80](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L80-L80)

```solidity
File: src/abstract/RewardSplits.sol

44               (paymentAmountWei * BUILDER_REWARD_BPS) /
45:              10_000 +

46               (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
47:              10_000 +

48               (paymentAmountWei * DEPLOYER_REWARD_BPS) /
49:              10_000 +

50               (paymentAmountWei * REVOLUTION_REWARD_BPS) /
51:              10_000;

57:                  builderReferralReward: (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000,

58:                  purchaseReferralReward: (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000,

59:                  deployerReward: (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000,

60:                  revolutionReward: (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000

```
*GitHub*: [44](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L44-L45), [46](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L46-L47), [48](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L48-L49), [50](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L50-L51), [57](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L57-L57), [58](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L58-L58), [59](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L59-L59), [60](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L60-L60)



### [G&#x2011;11] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function to save deployment gas
This will cost more runtime gas, but will reduce deployment gas when the function is (optionally called via a modifier) called more than once as is the case for the examples below. Most projects do not make this trade-off, but it's available nonetheless.

*There are 20 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

120:         require(msg.sender == address(manager), "Only manager can initialize");

222:         require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");

254:         require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

```
*GitHub*: [120](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L120-L120), [222](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L222-L222), [254](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L254-L254)

```solidity
File: src/CultureIndex.sol

117:         require(msg.sender == address(manager), "Only manager can initialize");

308:         require(pieceId < _currentPieceId, "Invalid piece ID");

452:         require(pieceId < _currentPieceId, "Invalid piece ID");

462:         require(pieceId < _currentPieceId, "Invalid piece ID");

```
*GitHub*: [117](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L117-L117), [308](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L308-L308), [452](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L452-L452), [462](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L462-L462)

```solidity
File: src/ERC20TokenEmitter.sol

91:          require(msg.sender == address(manager), "Only manager can initialize");

192:         require(success, "Transfer failed.");

197:             require(success, "Transfer failed.");

289:         require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

300:         require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");

```
*GitHub*: [91](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L91-L91), [192](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L192-L192), [197](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L197-L197), [289](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L289-L289), [300](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L300-L300)

```solidity
File: src/MaxHeap.sol

56:          require(msg.sender == address(manager), "Only manager can initialize");

157:         require(size > 0, "Heap is empty");

170:         require(size > 0, "Heap is empty");

```
*GitHub*: [56](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L56-L56), [157](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L157-L157), [170](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L170-L170)

```solidity
File: src/NontransferableERC20Votes.sol

69:          require(msg.sender == address(manager), "Only manager can initialize");

```
*GitHub*: [69](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L69-L69)

```solidity
File: src/VerbsToken.sol

137:         require(msg.sender == address(manager), "Only manager can initialize");

139:         require(_minter != address(0), "Minter cannot be zero address");

140:         require(_initialOwner != address(0), "Initial owner cannot be zero address");

210:         require(_minter != address(0), "Minter cannot be zero address");

```
*GitHub*: [137](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L137-L137), [139](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L139-L139), [140](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L140-L140), [210](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L210-L210)



### [G&#x2011;12] Events should be emitted outside of loops
Emitting an event has an overhead of **375 gas**, which will be incurred on every iteration of the loop. It is cheaper to `emit` only [once](https://github.com/ethereum/EIPs/blob/adad5968fd6de29902174e0cb51c8fc3dceb9ab5/EIPS/eip-1155.md?plain=1#L68) after the loop has finished.

*There is one instance of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit  _vote(), _voteForMany()
323:         emit VoteCast(pieceId, voter, weight, totalWeight);

```
*GitHub*: [323](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L323-L323)



### [G&#x2011;13] Multiple accesses of a `memory`/`calldata` array should use a local variable cache
The instances below point to the second+ access of a value inside a `memory`/`calldata` array, within a function. Caching avoids recalculating the array offsets into `memory`/`calldata`

*There is one instance of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit creatorArray...[]
187:             totalBps += creatorArray[i].bps;

```
*GitHub*: [187](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L187-L187)



### [G&#x2011;14] Reduce deployment costs by tweaking contracts' metadata
See [this](https://www.rareskills.io/post/solidity-metadata) link, at its bottom, for full details

*There are 7 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/AuctionHouse.sol

39   contract AuctionHouse is
40       IAuctionHouse,
41       VersionedContract,
42       UUPS,
43       PausableUpgradeable,
44       ReentrancyGuardUpgradeable,
45       Ownable2StepUpgradeable
46   {
47:      // The Verbs ERC721 token contract

```
*GitHub*: [39](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L39-L47)

```solidity
File: src/CultureIndex.sol

20   contract CultureIndex is
21       ICultureIndex,
22       VersionedContract,
23       UUPS,
24       Ownable2StepUpgradeable,
25       ReentrancyGuardUpgradeable,
26       EIP712Upgradeable
27   {
28:      /// @notice The EIP-712 typehash for gasless votes

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L20-L28)

```solidity
File: src/ERC20TokenEmitter.sol

17   contract ERC20TokenEmitter is
18       IERC20TokenEmitter,
19       ReentrancyGuardUpgradeable,
20       TokenEmitterRewards,
21       Ownable2StepUpgradeable,
22       PausableUpgradeable
23   {
24:      // treasury address to pay funds to

```
*GitHub*: [17](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L17-L24)

```solidity
File: src/MaxHeap.sol

14   contract MaxHeap is VersionedContract, UUPS, Ownable2StepUpgradeable, ReentrancyGuardUpgradeable {
15:      /// @notice The parent contract that is allowed to update the data store

```
*GitHub*: [14](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L14-L15)

```solidity
File: src/NontransferableERC20Votes.sol

29:  contract NontransferableERC20Votes is Initializable, ERC20VotesUpgradeable, Ownable2StepUpgradeable {

```
*GitHub*: [29](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L29-L29)

```solidity
File: src/VerbsToken.sol

33   contract VerbsToken is
34       IVerbsToken,
35       VersionedContract,
36       UUPS,
37       Ownable2StepUpgradeable,
38       ReentrancyGuardUpgradeable,
39       ERC721CheckpointableUpgradeable
40   {
41:      // An address who has permissions to mint Verbs

```
*GitHub*: [33](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L33-L41)

```solidity
File: src/libs/VRGDAC.sol

11   contract VRGDAC {
12       /*//////////////////////////////////////////////////////////////
13                               VRGDA PARAMETERS
14       //////////////////////////////////////////////////////////////*/
15:  

```
*GitHub*: [11](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L11-L15)

</details>





### [G&#x2011;15] Split `revert` checks to save gas
Splitting the conditions into two separate checks [saves](https://gist.github.com/IllIllI000/7e25b0fca6bd9d57d9b9bcb9d2018959) **2 gas**

*There are 2 instances of this issue:*

```solidity
File: src/CultureIndex.sol

441:         if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

```
*GitHub*: [441](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L441-L441)

```solidity
File: src/abstract/RewardSplits.sol

41:          if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

```
*GitHub*: [41](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41-L41)



### [G&#x2011;16] Use internal functions inside modifiers to save gas
This will cost more runtime gas, but will reduce deployment gas when the modifier is called more than once as is the case for the examples below. Most projects do not make this trade-off, but it's available nonetheless.

*There are 5 instances of this issue:*

```solidity
File: src/MaxHeap.sol

41       modifier onlyAdmin() {
42           require(msg.sender == admin, "Sender is not the admin");
43           _;
44:      }

```
*GitHub*: [41](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L41-L44)

```solidity
File: src/VerbsToken.sol

75       modifier whenMinterNotLocked() {
76           require(!isMinterLocked, "Minter is locked");
77           _;
78:      }

83       modifier whenCultureIndexNotLocked() {
84           require(!isCultureIndexLocked, "CultureIndex is locked");
85           _;
86:      }

91       modifier whenDescriptorNotLocked() {
92           require(!isDescriptorLocked, "Descriptor is locked");
93           _;
94:      }

99       modifier onlyMinter() {
100          require(msg.sender == minter, "Sender is not the minter");
101          _;
102:     }

```
*GitHub*: [75](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L75-L78), [83](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L83-L86), [91](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L91-L94), [99](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L99-L102)



### [G&#x2011;17] Using `private` rather than `public`, saves gas
For constants, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 44 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

48:      IVerbsToken public verbs;

51:      IERC20TokenEmitter public erc20TokenEmitter;

54:      address public WETH;

57:      uint256 public timeBuffer;

60:      uint256 public reservePrice;

63:      uint8 public minBidIncrementPercentage;

66:      uint256 public creatorRateBps;

69:      uint256 public minCreatorRateBps;

72:      uint256 public entropyRateBps;

75:      uint256 public duration;

78:      IAuctionHouse.Auction public auction;

```
*GitHub*: [48](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L48-L48), [51](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L51-L51), [54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L54-L54), [57](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L57-L57), [60](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L60-L60), [63](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L63-L63), [66](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L66-L66), [69](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L69-L69), [72](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L72-L72), [75](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L75-L75), [78](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L78-L78)

```solidity
File: src/CultureIndex.sol

33:      mapping(address => uint256) public nonces;

36:      MaxHeap public maxHeap;

39:      ERC20VotesUpgradeable public erc20VotingToken;

42:      ERC721CheckpointableUpgradeable public erc721VotingToken;

45:      uint256 public erc721VotingTokenWeight;

51:      uint256 public minVoteWeight;

54:      uint256 public quorumVotesBPS;

60:      string public description;

63:      mapping(uint256 => ArtPiece) public pieces;

66:      uint256 public _currentPieceId;

69:      mapping(uint256 => mapping(address => Vote)) public votes;

72:      mapping(uint256 => uint256) public totalVoteWeights;

78:      address public dropperAdmin;

```
*GitHub*: [33](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L33-L33), [36](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L36-L36), [39](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L39-L39), [42](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L42-L42), [45](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L45-L45), [51](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L51-L51), [54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L54-L54), [60](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L60-L60), [63](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L63-L63), [66](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L66-L66), [69](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L69-L69), [72](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L72-L72), [78](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L78-L78)

```solidity
File: src/ERC20TokenEmitter.sol

25:      address public treasury;

28:      NontransferableERC20Votes public token;

31:      VRGDAC public vrgdac;

34:      uint256 public startTime;

39:      int256 public emittedTokenWad;

42:      uint256 public creatorRateBps;

45:      uint256 public entropyRateBps;

48:      address public creatorsAddress;

```
*GitHub*: [25](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L25-L25), [28](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L28-L28), [31](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L31-L31), [34](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L34-L34), [39](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L39-L39), [42](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L42-L42), [45](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L45-L45), [48](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L48-L48)

```solidity
File: src/MaxHeap.sol

16:      address public admin;

65:      mapping(uint256 => uint256) public heap;

67:      uint256 public size = 0;

70:      mapping(uint256 => uint256) public valueMapping;

73:      mapping(uint256 => uint256) public positionMapping;

```
*GitHub*: [16](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L16-L16), [65](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L65-L65), [67](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L67-L67), [70](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L70-L70), [73](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L73-L73)

```solidity
File: src/VerbsToken.sol

42:      address public minter;

45:      IDescriptorMinimal public descriptor;

48:      ICultureIndex public cultureIndex;

51:      bool public isMinterLocked;

54:      bool public isCultureIndexLocked;

57:      bool public isDescriptorLocked;

66:      mapping(uint256 => ICultureIndex.ArtPiece) public artPieces;

```
*GitHub*: [42](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L42-L42), [45](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L45-L45), [48](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L48-L48), [51](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L51-L51), [54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L54-L54), [57](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L57-L57), [66](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L66-L66)
