## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Avoid contract existence checks by using low level calls | 5 |
| [GAS-2](#GAS-2) | Cache state variables to save gas | 33 |
| [GAS-3](#GAS-3) | State variables should be cached in stack variables | 29 |
| [GAS-4](#GAS-4) | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct | 1 |
| [GAS-5](#GAS-5) | Function Call Caching | 43 |
| [GAS-6](#GAS-6) | State variables only set in the constructor should be declared immutable | 7 |
| [GAS-7](#GAS-7) | Optimize gas costs using >= and <= operators | 66 |
| [GAS-8](#GAS-8) | Using private for constants | 5 |
| [GAS-9](#GAS-9) | State variables should be cached in stack variables | 29 |
| [GAS-10](#GAS-10) | Upgrade to Solidity 0.8.19 or Later for Gas Optimization | 9 |
| [GAS-11](#GAS-11) | Use assembly to emit events, in order to save gas | 28 |
| [GAS-12](#GAS-12) | Use assembly to write address/contract type storage values | 120 |
| [GAS-13](#GAS-13) | Using storage instead of memory for structs/arrays saves gas | 5 |
### <a name="GAS-1"></a>[GAS-1] Avoid contract existence checks by using low level calls

#### Impact:
Using low level calls can help avoid unnecessary contract existence checks, resulting in lower gas costs.

*Instances (5)*:
```solidity
File: scope/AuctionHouse.sol

370:                 uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;

371:                 address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

385:                         ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];

```

```solidity
File: scope/CultureIndex.sol

230:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();

```

```solidity
File: scope/VerbsToken.sol

282:         ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

```

### <a name="GAS-2"></a>[GAS-2] Cache state variables to save gas

#### Impact:
Caching state variables in stack variables can save gas by reducing read operations.

*Instances (33)*:
```solidity
File: scope/AuctionHouse.sol

96:         manager = IRevolutionBuilder(_manager);

134:         verbs = IVerbsToken(_erc721Token);

135:         erc20TokenEmitter = IERC20TokenEmitter(_erc20TokenEmitter);

188:         auction.bidder = payable(bidder);

317:             auction = Auction({

438:             bool wethSuccess = IWETH(WETH).transfer(_to, _amount);

```

```solidity
File: scope/CultureIndex.sol

29:     bytes32 public constant VOTE_TYPEHASH =

93:         manager = IRevolutionBuilder(_manager);

132:         erc20VotingToken = ERC20VotesUpgradeable(_erc20VotingToken);

133:         erc721VotingToken = ERC721CheckpointableUpgradeable(_erc721VotingToken);

144:         maxHeap = MaxHeap(_maxHeap);

213:         uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

226:         newPiece.totalVotesSupply = _calculateVoteWeight(

313:         uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);

375:         bool success = _verifyVoteSignature(from, pieceIds, deadline, v, r, s);

431:         voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

433:         bytes32 digest = _hashTypedDataV4(voteHash);

435:         address recoveredAddress = ecrecover(digest, v, r, s);

522:         ICultureIndex.ArtPiece memory piece = getTopVotedPiece();

```

```solidity
File: scope/ERC20TokenEmitter.sol

69:         manager = IRevolutionBuilder(_manager);

103:         vrgdac = VRGDAC(_vrgdac);

104:         token = NontransferableERC20Votes(_erc20Token);

165:         uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(

```

```solidity
File: scope/MaxHeap.sol

31:         manager = IRevolutionBuilder(_manager);

127:             current = parent(current);

148:                 position = parent(position);

```

```solidity
File: scope/NontransferableERC20Votes.sol

45:         manager = IRevolutionBuilder(_manager);

```

```solidity
File: scope/VRGDAC.sol

35:         decayConstant = wadLn(1e18 - _priceDecayPercent);

55:         int256 soldDifference = wadMul(perTimeUnit, timeSinceStart) - sold;

```

```solidity
File: scope/VerbsToken.sol

117:         manager = IRevolutionBuilder(_manager);

154:         descriptor = IDescriptorMinimal(_descriptor);

155:         cultureIndex = ICultureIndex(_cultureIndex);

```

```solidity
File: scope/abstract/RewardSplits.sol

32:         protocolRewards = IRevolutionProtocolRewards(_protocolRewards);

```

### <a name="GAS-3"></a>[GAS-3] State variables should be cached in stack variables

#### Impact:
Re-reading state variables from storage during a function call can result in higher gas costs. Caching state variables in stack variables can be a more gas-efficient approach.

*Instances (29)*:
```solidity
File: scope/AuctionHouse.sol

136:         timeBuffer = _auctionParams.timeBuffer;

137:         reservePrice = _auctionParams.reservePrice;

138:         minBidIncrementPercentage = _auctionParams.minBidIncrementPercentage;

139:         duration = _auctionParams.duration;

140:         creatorRateBps = _auctionParams.creatorRateBps;

141:         entropyRateBps = _auctionParams.entropyRateBps;

142:         minCreatorRateBps = _auctionParams.minCreatorRateBps;

185:         address payable lastBidder = _auction.bidder;

187:         auction.amount = msg.value;

314:             uint256 startTime = block.timestamp;

```

```solidity
File: scope/CultureIndex.sol

134:         erc721VotingTokenWeight = _cultureIndexParams.erc721VotingTokenWeight;

135:         name = _cultureIndexParams.name;

136:         description = _cultureIndexParams.description;

137:         quorumVotesBPS = _cultureIndexParams.quorumVotesBPS;

138:         minVoteWeight = _cultureIndexParams.minVoteWeight;

180:         uint256 creatorArrayLength = creatorArray.length;

232:         newPiece.sponsor = msg.sender;

233:         newPiece.creationBlock = block.number;

354:         uint256 len = pieceIds.length;

397:         uint256 len = from.length;

```

```solidity
File: scope/ERC20TokenEmitter.sol

105:         startTime = block.timestamp;

```

```solidity
File: scope/VerbsToken.sol

150:         _contractURIHash = _erc721TokenParams.contractURIHash;

298:             newPiece.pieceId = artPiece.pieceId;

299:             newPiece.metadata = artPiece.metadata;

300:             newPiece.isDropped = artPiece.isDropped;

301:             newPiece.sponsor = artPiece.sponsor;

302:             newPiece.totalERC20Supply = artPiece.totalERC20Supply;

303:             newPiece.quorumVotes = artPiece.quorumVotes;

304:             newPiece.totalVotesSupply = artPiece.totalVotesSupply;

```

### <a name="GAS-4"></a>[GAS-4] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct

#### Impact:
Combining multiple mappings into a single mapping of an address/ID to a struct can save storage slots, reduce gas costs, and optimize contract storage.

*Instances (1)*:
```solidity
File: scope/CultureIndex.sol

33:     mapping(address => uint256) public nonces;

```

### <a name="GAS-5"></a>[GAS-5] Function Call Caching

#### Impact:
Caching function results reduces gas consumption by avoiding repeated calls.

*Instances (43)*:
```solidity
File: scope/AuctionHouse.sol

313:         try verbs.mint() returns (uint256 verbId) {

355:             verbs.burn(_auction.verbId);

359:                 verbs.burn(_auction.verbId);

361:             else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

370:                 uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;

371:                 address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

385:                         ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];

403:                         IERC20TokenEmitter.ProtocolRewardAddresses({

454:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```

```solidity
File: scope/CultureIndex.sol

128:         __EIP712_init(string.concat(_cultureIndexParams.name, " CultureIndex"), "1");

221:         maxHeap.insert(pieceId, 0);

227:             erc20VotingToken.totalSupply(),

228:             erc721VotingToken.totalSupply()

230:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();

237:             newPiece.creators.push(creatorArray[i]);

289:         return _calculateVoteWeight(erc20VotingToken.getVotes(account), erc721VotingToken.getVotes(account));

289:         return _calculateVoteWeight(erc20VotingToken.getVotes(account), erc721VotingToken.getVotes(account));

295:                 erc20VotingToken.getPastVotes(account, blockNumber),

296:                 erc721VotingToken.getPastVotes(account, blockNumber)

322:         maxHeap.updateValue(pieceId, totalWeight);

431:         voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

487:         require(maxHeap.size() > 0, "Culture index is empty");

489:         (uint256 pieceId, ) = maxHeap.getMax();

511:             (quorumVotesBPS * _calculateVoteWeight(erc20VotingToken.totalSupply(), erc721VotingToken.totalSupply())) /

511:             (quorumVotesBPS * _calculateVoteWeight(erc20VotingToken.totalSupply(), erc721VotingToken.totalSupply())) /

529:         maxHeap.extractMax();

545:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```

```solidity
File: scope/ERC20TokenEmitter.sol

109:         token.mint(_to, _amount);

114:         return token.totalSupply();

119:         return token.decimals();

124:         return token.balanceOf(_owner);

242:             vrgdac.xToY({

259:             vrgdac.yToX({

276:             vrgdac.yToX({

```

```solidity
File: scope/MaxHeap.sol

183:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```

```solidity
File: scope/VerbsToken.sol

162:         return string(abi.encodePacked("ipfs://", _contractURIHash));

194:         return descriptor.tokenURI(tokenId, artPieces[tokenId].metadata);

202:         return descriptor.dataURI(tokenId, artPieces[tokenId].metadata);

282:         ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

287:             artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),

292:         try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {

307:                 newPiece.creators.push(artPiece.creators[i]);

330:         require(manager.isRegisteredUpgrade(_getImplementation(), _newImpl), "Invalid upgrade");

```

### <a name="GAS-6"></a>[GAS-6] State variables only set in the constructor should be declared immutable

#### Impact:
Avoids a Gsset (20000 gas) in the constructor and reduces gas costs for state variable accesses.

*Instances (7)*:
```solidity
File: scope/AuctionHouse.sol

95:     constructor(address _manager) payable initializer {

```

```solidity
File: scope/CultureIndex.sol

92:     constructor(address _manager) payable initializer {

```

```solidity
File: scope/ERC20TokenEmitter.sol

64:     constructor(

```

```solidity
File: scope/MaxHeap.sol

30:     constructor(address _manager) payable initializer {

```

```solidity
File: scope/VRGDAC.sol

28:     constructor(int256 _targetPrice, int256 _priceDecayPercent, int256 _perTimeUnit) {

```

```solidity
File: scope/VerbsToken.sol

116:     constructor(address _manager) payable initializer {

```

```solidity
File: scope/abstract/RewardSplits.sol

29:     constructor(address _protocolRewards, address _revolutionRewardRecipient) payable {

```

### <a name="GAS-7"></a>[GAS-7] Optimize gas costs using >= and <= operators

#### Impact:
Using >= and <= operators can save gas compared to > and < operators.

*Instances (66)*:
```solidity
File: scope/AuctionHouse.sol

130:             _auctionParams.creatorRateBps >= _auctionParams.minCreatorRateBps,

178:         require(block.timestamp < _auction.endTime, "Auction expired");

179:         require(msg.value >= reservePrice, "Must send at least reservePrice");

181:             msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),

191:         bool extended = _auction.endTime - block.timestamp < timeBuffer;

219:             _creatorRateBps >= minCreatorRateBps,

222:         require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");

234:         require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");

235:         require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");

239:             _minCreatorRateBps > minCreatorRateBps,

254:         require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

342:         require(block.timestamp >= _auction.endTime, "Auction hasn't completed");

348:         if (address(this).balance < reservePrice) {

363:             if (_auction.amount > 0) {

383:                 if (creatorsShare > 0 && entropyRateBps > 0) {

383:                 if (creatorsShare > 0 && entropyRateBps > 0) {

384:                     for (uint256 i = 0; i < numCreators; i++) {

399:                 if (creatorsShare > ethPaidToCreators) {

421:         if (address(this).balance < _amount) revert("Insufficient balance");

```

```solidity
File: scope/CultureIndex.sol

119:         require(_cultureIndexParams.quorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "invalid quorum bps");

120:         require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");

163:             require(bytes(metadata.image).length > 0, "Image URL must be provided");

165:             require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");

167:             require(bytes(metadata.text).length > 0, "Text must be provided");

182:         require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

182:         require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

185:         for (uint i; i < creatorArrayLength; i++) {

236:         for (uint i; i < creatorArrayLength; i++) {

243:         for (uint i; i < creatorArrayLength; i++) {

308:         require(pieceId < _currentPieceId, "Invalid piece ID");

314:         require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");

355:         for (uint256 i; i < len; i++) {

403:         for (uint256 i; i < len; i++) {

407:         for (uint256 i; i < len; i++) {

427:         require(deadline >= block.timestamp, "Signature expired");

452:         require(pieceId < _currentPieceId, "Invalid piece ID");

462:         require(pieceId < _currentPieceId, "Invalid piece ID");

499:         require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");

```

```solidity
File: scope/ERC20TokenEmitter.sol

160:         require(msg.value > 0, "Must send ether");

184:         int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);

188:         if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

195:         if (creatorDirectPayment > 0) {

201:         if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {

209:         for (uint256 i = 0; i < addresses.length; i++) {

210:             if (totalTokensForBuyers > 0) {

238:         require(amount > 0, "Amount must be greater than 0");

255:         require(etherAmount > 0, "Ether amount must be greater than 0");

272:         require(paymentAmount > 0, "Payment amount must be greater than 0");

289:         require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

300:         require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");

```

```solidity
File: scope/MaxHeap.sol

102:         if (pos >= (size / 2) && pos <= size) return;

104:         if (posValue < leftValue || posValue < rightValue) {

104:         if (posValue < leftValue || posValue < rightValue) {

105:             if (leftValue > rightValue) {

144:         if (newValue > oldValue) {

150:         } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify

157:         require(size > 0, "Heap is empty");

170:         require(size > 0, "Heap is empty");

```

```solidity
File: scope/VRGDAC.sol

38:         require(decayConstant < 0, "NON_NEGATIVE_DECAY_CONSTANT");

```

```solidity
File: scope/VerbsToken.sol

274:         require(verbId <= _currentVerbId, "Invalid piece ID");

287:             artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),

288:             "Creator array must not be > MAX_NUM_CREATORS"

306:             for (uint i = 0; i < artPiece.creators.length; i++) {

```

```solidity
File: scope/abstract/RewardSplits.sol

41:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

41:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

```

```solidity
File: scope/abstract/TokenEmitter/TokenEmitterRewards.sol

18:         if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();

```

### <a name="GAS-8"></a>[GAS-8] Using private for constants

#### Impact:
Using private for constants can save gas in deployment and is preferable for public constants.

*Instances (5)*:
```solidity
File: scope/AuctionHouse.sol

88:     uint32 public constant MIN_TOKEN_MINT_GAS_THRESHOLD = 750_000;

```

```solidity
File: scope/CultureIndex.sol

48:     uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%

75:     uint256 public constant MAX_NUM_CREATORS = 100;

```

```solidity
File: scope/abstract/RewardSplits.sol

23:     uint256 public constant minPurchaseAmount = 0.0000001 ether;

24:     uint256 public constant maxPurchaseAmount = 50_000 ether;

```

### <a name="GAS-9"></a>[GAS-9] State variables should be cached in stack variables

#### Impact:
Re-reading state variables from storage during a function call can result in higher gas costs. Caching state variables in stack variables can be a more gas-efficient approach.

*Instances (29)*:
```solidity
File: scope/AuctionHouse.sol

136:         timeBuffer = _auctionParams.timeBuffer;

137:         reservePrice = _auctionParams.reservePrice;

138:         minBidIncrementPercentage = _auctionParams.minBidIncrementPercentage;

139:         duration = _auctionParams.duration;

140:         creatorRateBps = _auctionParams.creatorRateBps;

141:         entropyRateBps = _auctionParams.entropyRateBps;

142:         minCreatorRateBps = _auctionParams.minCreatorRateBps;

185:         address payable lastBidder = _auction.bidder;

187:         auction.amount = msg.value;

314:             uint256 startTime = block.timestamp;

```

```solidity
File: scope/CultureIndex.sol

134:         erc721VotingTokenWeight = _cultureIndexParams.erc721VotingTokenWeight;

135:         name = _cultureIndexParams.name;

136:         description = _cultureIndexParams.description;

137:         quorumVotesBPS = _cultureIndexParams.quorumVotesBPS;

138:         minVoteWeight = _cultureIndexParams.minVoteWeight;

180:         uint256 creatorArrayLength = creatorArray.length;

232:         newPiece.sponsor = msg.sender;

233:         newPiece.creationBlock = block.number;

354:         uint256 len = pieceIds.length;

397:         uint256 len = from.length;

```

```solidity
File: scope/ERC20TokenEmitter.sol

105:         startTime = block.timestamp;

```

```solidity
File: scope/VerbsToken.sol

150:         _contractURIHash = _erc721TokenParams.contractURIHash;

298:             newPiece.pieceId = artPiece.pieceId;

299:             newPiece.metadata = artPiece.metadata;

300:             newPiece.isDropped = artPiece.isDropped;

301:             newPiece.sponsor = artPiece.sponsor;

302:             newPiece.totalERC20Supply = artPiece.totalERC20Supply;

303:             newPiece.quorumVotes = artPiece.quorumVotes;

304:             newPiece.totalVotesSupply = artPiece.totalVotesSupply;

```

### <a name="GAS-10"></a>[GAS-10] Upgrade to Solidity 0.8.19 or Later for Gas Optimization

#### Impact:
Upgrading to Solidity 0.8.19 or a later version may optimize gas usage.

*Instances (9)*:
```solidity
File: scope/AuctionHouse.sol

24: pragma solidity ^0.8.22;

```

```solidity
File: scope/CultureIndex.sol

2: pragma solidity ^0.8.22;

```

```solidity
File: scope/ERC20TokenEmitter.sol

2: pragma solidity ^0.8.22;

```

```solidity
File: scope/MaxHeap.sol

2: pragma solidity ^0.8.22;

```

```solidity
File: scope/NontransferableERC20Votes.sol

4: pragma solidity ^0.8.22;

```

```solidity
File: scope/VRGDAC.sol

2: pragma solidity 0.8.22;

```

```solidity
File: scope/VerbsToken.sol

18: pragma solidity ^0.8.22;

```

```solidity
File: scope/abstract/RewardSplits.sol

2: pragma solidity 0.8.22;

```

```solidity
File: scope/abstract/TokenEmitter/TokenEmitterRewards.sol

2: pragma solidity 0.8.22;

```

### <a name="GAS-11"></a>[GAS-11] Use assembly to emit events, in order to save gas

#### Impact:
Using assembly to emit events can save gas by making use of scratch space and free memory pointers, avoiding memory expansion costs.

*Instances (28)*:
```solidity
File: scope/AuctionHouse.sol

197:         emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);

199:         if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);

225:         emit CreatorRateBpsUpdated(_creatorRateBps);

245:         emit MinCreatorRateBpsUpdated(_minCreatorRateBps);

257:         emit EntropyRateBpsUpdated(_entropyRateBps);

280:         emit AuctionTimeBufferUpdated(_timeBuffer);

290:         emit AuctionReservePriceUpdated(_reservePrice);

300:         emit AuctionMinBidIncrementPercentageUpdated(_minBidIncrementPercentage);

326:             emit AuctionCreated(verbId, startTime, endTime);

413:         emit AuctionSettled(_auction.verbId, _auction.bidder, _auction.amount, creatorTokensEmitted);

```

```solidity
File: scope/CultureIndex.sol

141:         emit QuorumVotesBPSSet(quorumVotesBPS, _cultureIndexParams.quorumVotesBPS);

240:         emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

244:             emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);

323:         emit VoteCast(pieceId, voter, weight, totalWeight);

500:         emit QuorumVotesBPSSet(quorumVotesBPS, newQuorumVotesBPS);

531:         emit PieceDropped(piece.pieceId, msg.sender);

```

```solidity
File: scope/ERC20TokenEmitter.sol

219:         emit PurchaseFinalized(

291:         emit EntropyRateBpsUpdated(entropyRateBps = _entropyRateBps);

302:         emit CreatorRateBpsUpdated(creatorRateBps = _creatorRateBps);

312:         emit CreatorsAddressUpdated(creatorsAddress = _creatorsAddress);

```

```solidity
File: scope/VerbsToken.sol

186:         emit VerbBurned(verbId);

213:         emit MinterUpdated(_minter);

223:         emit MinterLocked();

235:         emit DescriptorUpdated(_descriptor);

245:         emit DescriptorLocked();

255:         emit CultureIndexUpdated(_cultureIndex);

265:         emit CultureIndexLocked();

312:             emit VerbCreated(verbId, artPiece);

```

### <a name="GAS-12"></a>[GAS-12] Use assembly to write address/contract type storage values

#### Impact:
Using assembly to write address or contract type storage values can save gas.

*Instances (120)*:
```solidity
File: scope/AuctionHouse.sol

88:     uint32 public constant MIN_TOKEN_MINT_GAS_THRESHOLD = 750_000;

96:         manager = IRevolutionBuilder(_manager);

134:         verbs = IVerbsToken(_erc721Token);

135:         erc20TokenEmitter = IERC20TokenEmitter(_erc20TokenEmitter);

136:         timeBuffer = _auctionParams.timeBuffer;

137:         reservePrice = _auctionParams.reservePrice;

138:         minBidIncrementPercentage = _auctionParams.minBidIncrementPercentage;

139:         duration = _auctionParams.duration;

140:         creatorRateBps = _auctionParams.creatorRateBps;

141:         entropyRateBps = _auctionParams.entropyRateBps;

142:         minCreatorRateBps = _auctionParams.minCreatorRateBps;

143:         WETH = _weth;

172:         IAuctionHouse.Auction memory _auction = auction;

185:         address payable lastBidder = _auction.bidder;

187:         auction.amount = msg.value;

188:         auction.bidder = payable(bidder);

223:         creatorRateBps = _creatorRateBps;

243:         minCreatorRateBps = _minCreatorRateBps;

256:         entropyRateBps = _entropyRateBps;

278:         timeBuffer = _timeBuffer;

288:         reservePrice = _reservePrice;

298:         minBidIncrementPercentage = _minBidIncrementPercentage;

314:             uint256 startTime = block.timestamp;

337:         IAuctionHouse.Auction memory _auction = auction;

344:         auction.settled = true;

346:         uint256 creatorTokensEmitted = 0;

370:                 uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;

371:                 address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

380:                 uint256 ethPaidToCreators = 0;

384:                     for (uint256 i = 0; i < numCreators; i++) {

385:                         ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];

```

```solidity
File: scope/CultureIndex.sol

48:     uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%

75:     uint256 public constant MAX_NUM_CREATORS = 100;

93:         manager = IRevolutionBuilder(_manager);

132:         erc20VotingToken = ERC20VotesUpgradeable(_erc20VotingToken);

133:         erc721VotingToken = ERC721CheckpointableUpgradeable(_erc721VotingToken);

134:         erc721VotingTokenWeight = _cultureIndexParams.erc721VotingTokenWeight;

135:         name = _cultureIndexParams.name;

136:         description = _cultureIndexParams.description;

137:         quorumVotesBPS = _cultureIndexParams.quorumVotesBPS;

138:         minVoteWeight = _cultureIndexParams.minVoteWeight;

139:         dropperAdmin = _dropperAdmin;

144:         maxHeap = MaxHeap(_maxHeap);

180:         uint256 creatorArrayLength = creatorArray.length;

213:         uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

218:         uint256 pieceId = _currentPieceId++;

223:         ArtPiece storage newPiece = pieces[pieceId];

225:         newPiece.pieceId = pieceId;

230:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();

231:         newPiece.metadata = metadata;

232:         newPiece.sponsor = msg.sender;

233:         newPiece.creationBlock = block.number;

319:         uint256 totalWeight = totalVoteWeights[pieceId];

354:         uint256 len = pieceIds.length;

397:         uint256 len = from.length;

433:         bytes32 digest = _hashTypedDataV4(voteHash);

502:         quorumVotesBPS = newQuorumVotesBPS;

522:         ICultureIndex.ArtPiece memory piece = getTopVotedPiece();

526:         pieces[piece.pieceId].isDropped = true;

```

```solidity
File: scope/ERC20TokenEmitter.sol

69:         manager = IRevolutionBuilder(_manager);

101:         treasury = _treasury;

102:         creatorsAddress = _creatorsAddress;

103:         vrgdac = VRGDAC(_vrgdac);

104:         token = NontransferableERC20Votes(_erc20Token);

105:         startTime = block.timestamp;

205:         uint256 bpsSum = 0;

209:         for (uint256 i = 0; i < addresses.length; i++) {

291:         emit EntropyRateBpsUpdated(entropyRateBps = _entropyRateBps);

302:         emit CreatorRateBpsUpdated(creatorRateBps = _creatorRateBps);

312:         emit CreatorsAddressUpdated(creatorsAddress = _creatorsAddress);

```

```solidity
File: scope/MaxHeap.sol

31:         manager = IRevolutionBuilder(_manager);

58:         admin = _admin;

67:     uint256 public size = 0;

98:         uint256 posValue = valueMapping[heap[pos]];

99:         uint256 leftValue = valueMapping[heap[left]];

100:         uint256 rightValue = valueMapping[heap[right]];

124:         uint256 current = size;

127:             current = parent(current);

137:         uint256 position = positionMapping[itemId];

138:         uint256 oldValue = valueMapping[itemId];

148:                 position = parent(position);

159:         uint256 popped = heap[0];

```

```solidity
File: scope/NontransferableERC20Votes.sol

45:         manager = IRevolutionBuilder(_manager);

```

```solidity
File: scope/VRGDAC.sol

29:         targetPrice = _targetPrice;

31:         perTimeUnit = _perTimeUnit;

33:         priceDecayPercent = _priceDecayPercent;

```

```solidity
File: scope/VerbsToken.sol

63:     string private _contractURIHash = "QmQzDwaZ7yQxHHs7sQQenJVB89riTSacSGcJRv9jtHPuz5";

117:         manager = IRevolutionBuilder(_manager);

150:         _contractURIHash = _erc721TokenParams.contractURIHash;

153:         minter = _minter;

154:         descriptor = IDescriptorMinimal(_descriptor);

155:         cultureIndex = ICultureIndex(_cultureIndex);

170:         _contractURIHash = newContractURIHash;

211:         minter = _minter;

221:         isMinterLocked = true;

233:         descriptor = _descriptor;

243:         isDescriptorLocked = true;

253:         cultureIndex = _cultureIndex;

263:         isCultureIndexLocked = true;

282:         ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

293:             artPiece = _artPiece;

294:             uint256 verbId = _currentVerbId++;

296:             ICultureIndex.ArtPiece storage newPiece = artPieces[verbId];

298:             newPiece.pieceId = artPiece.pieceId;

299:             newPiece.metadata = artPiece.metadata;

300:             newPiece.isDropped = artPiece.isDropped;

301:             newPiece.sponsor = artPiece.sponsor;

302:             newPiece.totalERC20Supply = artPiece.totalERC20Supply;

303:             newPiece.quorumVotes = artPiece.quorumVotes;

304:             newPiece.totalVotesSupply = artPiece.totalVotesSupply;

306:             for (uint i = 0; i < artPiece.creators.length; i++) {

```

```solidity
File: scope/abstract/RewardSplits.sol

18:     uint256 internal constant DEPLOYER_REWARD_BPS = 25;

19:     uint256 internal constant REVOLUTION_REWARD_BPS = 75;

20:     uint256 internal constant BUILDER_REWARD_BPS = 100;

21:     uint256 internal constant PURCHASE_REFERRAL_BPS = 50;

32:         protocolRewards = IRevolutionProtocolRewards(_protocolRewards);

33:         revolutionRewardRecipient = _revolutionRewardRecipient;

74:         if (builderReferral == address(0)) builderReferral = revolutionRewardRecipient;

76:         if (deployer == address(0)) deployer = revolutionRewardRecipient;

78:         if (purchaseReferral == address(0)) purchaseReferral = revolutionRewardRecipient;

```

### <a name="GAS-13"></a>[GAS-13] Using storage instead of memory for structs/arrays saves gas

#### Impact:
Fetching data from a storage location and assigning it to a memory variable can result in higher gas costs. Using storage variables directly can be more gas-efficient.

*Instances (5)*:
```solidity
File: scope/AuctionHouse.sol

172:         IAuctionHouse.Auction memory _auction = auction;

337:         IAuctionHouse.Auction memory _auction = auction;

385:                         ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];

```

```solidity
File: scope/CultureIndex.sol

522:         ICultureIndex.ArtPiece memory piece = getTopVotedPiece();

```

```solidity
File: scope/VerbsToken.sol

282:         ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

```

