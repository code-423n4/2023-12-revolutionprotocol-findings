## Gas Optimizations

| Number                                                                                                              | Issue                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------- |
| [[G-01](#g-01-state-variables-can-be-packed-into-fewer-storage-slot-saves-12000-gas)]                               | State variables can be packed into fewer storage slot (saves 12000 Gas)                               |
| [[G-02](#g-02-pack-structs-by-putting-variables-that-can-fit-together-next-to-each-other-saves-8000-gas)]           | Pack structs by putting variables that can fit together next to each other (saves 8000 Gas)           |
| [[G-03](#g-03-unnecessary-modifier-checks-in-constructors-saves-144000-gas)]                                        | Unnecessary `modifier` checks in constructors (saves 144000 GAS)                                      |
| [[G-04](#g-04-unnecessary-modifier-checks-in-functions)]                                                            | Unnecessary `modifier` checks in functions(saves 123500 GAS)                                          |
| [[G-05](#g-05-remove-vote-struct-as-value-from-votes-mapping-in-place-of-that-use-only-uin256-weight-as-value)]     | Remove `Vote` struct as value from `votes` mapping in place of that use only `uin256 weight` as value |
| [[G-06](#g-06-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage)]        | State variables should be cached in stack variables rather than re-reading them from storage          |
| [[G-07](#g-07-state-address-variable-which-is-fix-for-chain-can-be-marked-immutable-and-initialize-in-constructor)] | State address variable which is fix for chain can be marked immutable and initialize in constructor   |
| [[G-08](#g-08-do-not-assign-struct-value-second-time-if-it-is-already-assigned-it-saves-multiple-mload-and-mstore)] | Do not assign struct value second time if it is already assigned it saves multiple MLOAD and MSTORE   |
| [[G-09](#g-09-using-storage-instead-of-memory-for-structsarrays-saves-gas-saves-4200-gas)]                          | Using `storage` instead of memory for structs/arrays saves gas (saves 4200 GAS)                       |
| [[G-10](#g-10-refactor-code-to-fail-early)]                                                                         | Refactor code to fail early                                                                           |
| [[G-11](#g-11-refactor-the-computetotalreward-function-to-save-gas)]                                                | Refactor the `computeTotalReward` function to save gas                                                |
| [[G-12](#g-12-refactor-the-_vote-function-to-save-gas)]                                                             | Refactor the `_vote` function to save gas                                                             |
| [[G-13](#g-13-remove-unnecessary-checks)]                                                                           | Remove unnecessary checks                                                                             |
| [[G-14](#g-14-missing-zero-address-check-in-constructor)]                                                           | Missing zero-address check in constructor                                                             |

## [G-01] State variables can be packed into fewer storage slot (saves 12000 Gas)

### `SAVE: 12000 GAS, 6 SLOT`

### NOTE: Not in bot report

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

### `creatorRateBps`, `minCreatorRateBps`, `entropyRateBps` and `address WETH` can be packed in single SLOT :`SAVES 6000 GAS`, `3 SLOT`

`creatorRateBps`, `minCreatorRateBps`, and `entropyRateBps` can't be more than 10000 because of these checks :

```solidity
File : main/packages/revolution/src/AuctionHouse.sol

218:    require(
219:        _creatorRateBps >= minCreatorRateBps,
220:         "Creator rate must be greater than or equal to minCreatorRateBps"
221:       );
222:    require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");

254:    require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

```

[L218-L222](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L218C1-L223C1), [L254](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L254)

So uint16 is more than sufficient to hold `creatorRateBps`, `minCreatorRateBps`, and `entropyRateBps` because these value can't more than 10000 and uint16 can easily hold up to 0 to 65535. All 3 Bps will take 6 bytes in slot so these can be packed with address because address take 20 byte space in 32 byte storage slot.

```diff
File : main/packages/revolution/src/AuctionHouse.sol

54:    address public WETH;
+       uint16 public creatorRateBps;

+       uint16 public minCreatorRateBps;

+       uint16 public entropyRateBps;

55:
56:    // The minimum amount of time left in an auction after a new bid is created
57:    uint256 public timeBuffer;
58:
59:    // The minimum price accepted in an auction
60:    uint256 public reservePrice;
61:
62:    // The minimum percentage difference between the last bid amount and the current bid
63:    uint8 public minBidIncrementPercentage;
64:
65:    // The split of the winning bid that is reserved for the creator of the Verb in basis points
-66:    uint256 public creatorRateBps;
67:
68:    // The all time minimum split of the winning bid that is reserved for the creator of the Verb in basis points
-69:    uint256 public minCreatorRateBps;
70:
71:    // The split of (auction proceeds * creatorRate) that is sent to the creator as ether in basis points
-72:    uint256 public entropyRateBps;

```

[L54-L72](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L54C1-L72C35)

### `quorumVotesBPS` and `dropperAdmin` can be packed to single SLOT :`SAVES 2000 GAS, 1 SLOT`

`quorumVotesBPS` can't more than 6000 because of these checks:

```solidity
File : main/packages/revolution/src/CultureIndex.sol

48: uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%

498: function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external onlyOwner {
499:    require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");

```

[L498-L499](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L498C5-L499C116), [L48](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L48)

Here `quorumVotesBPS` can't more than MAX_QUORUM_VOTES_BPS and MAX_QUORUM_VOTES_BPS is 6000. So `quorumVotesBPS` can be easily reduced to uint16 and pack with an address `dropperAdmin` because address take up space of 20 bytes in a 32 byte storage slot. So `quorumVotesBPS` can easily pack with `dropperAdmin`.

```diff
File: packages/revolution/src/CultureIndex.sol

-54: uint256 public quorumVotesBPS;
    ...
78: address public dropperAdmin;
+54: uint16 public quorumVotesBPS;

```

[L54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L54), [L78](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L78)

### `creatorRateBps`, `entropyRateBps` and `creatorsAddress` can be packed in single SLOT :`SAVES 4000 GAS, 2 SLOT`

`creatorRateBps` and `entropyRateBps` can't more than 10000 because of these checks:

```solidity
File : main/packages/revolution/src/ERC20TokenEmitter.sol

289:    require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

300:    require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
```

[L289](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L289), [L300](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L300)

So uint16 is more than sufficient to hold `creatorRateBps`, `entropyRateBps` because these value can't more than 10000 and uint16 can easily hold up to 0 to 65535. `creatorRateBps` and `entropyRateBps` can be packed with address because address take 20 byte space in 32 byte storage slot.

```diff
File : main/packages/revolution/src/ERC20TokenEmitter.sol

-42:    uint256 public creatorRateBps;
+42:    uint16 public creatorRateBps;

-45:    uint256 public entropyRateBps;
+45:    uint16 public entropyRateBps;

48:    // The account or contract to pay the creator reward to
    address public creatorsAddress;

```

[L42-L48](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L41C1-L48C36)

## [G-02] Pack structs by putting variables that can fit together next to each other (saves 8000 Gas)

### `SAVE: 8000 GAS, 4 SLOT`

### Note: From Contest ReadMe _Any issues or improvements on how we integrate with the out of scope contracts is in scope_.

Since interfaces directly not in scope but struct defined inside them used by in-scope contracts ie. variables defined in-scope contracts using those struct from interfaces. So these varaibles will take storage space inside in-scope. So to by packing struct in interfaces will save SLOTs and Gas inside in-scope contracts. So that's why I am including those interfaces struct packing which are inherited by in-scope contracts.

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas

### truncate `startTime` and `endTime` to uint40 and pack with `bidder` :`SAVES 4000 GAS, 2 SLOT`

The uint40 type is safe for approximately 1.099511627776e+12 years. The exact number of years that a uint40 type is safe for is 1099511627776 years.

```diff
File : main/packages/revolution/src/interfaces/IAuctionHouse.sol

23: struct Auction {
        // ID for the Verb (ERC721 token ID)
        uint256 verbId;
        // The current highest bid amount
        uint256 amount;
        // The time that the auction started
-       uint256 startTime;
        // The time that the auction is scheduled to end
-       uint256 endTime;
        // The address of the current highest bid
        address payable bidder;
+       uint40 startTime;
+       uint40 endTime;
        // Whether or not the auction has been settled
        bool settled;
36:    }

```

[L23-L36](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/interfaces/IAuctionHouse.sol#L23C5-L36C6)

### `bps` and `creator` can be packed in single SLOT :`SAVES 2000 GAS, 1 SLOT`

Reduce uint type for bps to `uint16` because bps can't more than 10000 so `uint16` is enough to hold it.

```diff
File : main/packages/revolution/src/interfaces/ICultureIndex.sol

98: struct CreatorBps {
99:        address creator;
+100:        uint16 bps;
-100:        uint256 bps;
101:    }

```

[L98-L101](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/interfaces/ICultureIndex.sol#L98C5-L101C6)

### `creationBlock` and `sponsor` can be packed in single SLOT :`SAVES 2000 GAS, 1 SLOT`

uint48 is more than sufficient to hold block number.

```diff
File : main/packages/revolution/src/interfaces/ICultureIndex.sol

115: struct ArtPiece {
        uint256 pieceId;
        ArtPieceMetadata metadata;
        CreatorBps[] creators;
        address sponsor;
        bool isDropped;
+        uint48 creationBlock;
-        uint256 creationBlock;
        uint256 quorumVotes;
        uint256 totalERC20Supply;
        uint256 totalVotesSupply;
125:    }

```

[L115-L125](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/interfaces/ICultureIndex.sol#L115C5-L125C6)

## [G-03] Unnecessary `modifier` checks in constructors (saves 144000 GAS)

### `SAVE: 144000 GAS on DEPLOYMENT`

### Unnecessary `initializer` check

Since initializer modifier is used to make initialize function run only once in upgradeable contracts. But constructor runs only once already at the time of deployment so there is no need of initializer modifier in constructor.

### `SAVES ~24000 GAS` on DEPLOYMENT

```solidity
File : main/packages/revolution/src/AuctionHouse.sol

95: constructor(address _manager) payable initializer {

```

[L95](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L95)

### `SAVES 24000 GAS` on DEPLOYMENT

```solidity
File : main/packages/revolution/src/CultureIndex.sol

92: constructor(address _manager) payable initializer {

```

[L92](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L92)

### `SAVES 24000 GAS` on DEPLOYMENT

```solidity
File : main/packages/revolution/src/ERC20TokenEmitter.sol

64: constructor(
65:        address _manager,
66:        address _protocolRewards,
67:        address _protocolFeeRecipient
68:    ) payable TokenEmitterRewards(_protocolRewards, _protocolFeeRecipient) initializer {
69:        manager = IRevolutionBuilder(_manager);
70:    }
```

[L64-L70](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L64C5-L70C6)

### `SAVES 24000 GAS` on DEPLOYMENT

```solidity
File : main/packages/revolution/src/MaxHeap.sol

30: constructor(address _manager) payable initializer {
31:        manager = IRevolutionBuilder(_manager);
32:    }

```

[L30-L32](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L30C5-L32C6)

### `SAVES 24000 GAS` on DEPLOYMENT

```solidity
File : main/packages/revolution/src/NontransferableERC20Votes.so

44: constructor(address _manager) payable initializer {
45:        manager = IRevolutionBuilder(_manager);
46:    }

```

[L44-L46](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L44C5-L46C6)

### `SAVES 24000 GAS` on DEPLOYMENT

```solidity
File : revolution/src/VerbsToken.sol

116: constructor(address _manager) payable initializer {
117:        manager = IRevolutionBuilder(_manager);
118:  }
```

[L116-L118](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L116C1-L118C6)

## [G-04] Unnecessary `modifier` checks in functions(saves 123500 GAS)

### Unnecessary `nonReentrant` check

Function not calling any external contract can't be in re-entrancy also function can only be called by onlyOwner/onlyMinter so no chance to re-enter here.

### `SAVES ~24700 GAS` when every time this function called

```solidity
File : main/packages/revolution/src/ERC20TokenEmitter.sol

309: function setCreatorsAddress(address _creatorsAddress) external override onlyOwner nonReentrant {
310:      require(_creatorsAddress != address(0), "Invalid address");
311:
312:        emit CreatorsAddressUpdated(creatorsAddress = _creatorsAddress);
313:    }

```

[L309-L313](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L309C5-L313C6)

### `SAVES ~24700 GAS` when every time this function called

```solidity
File : main/packages/revolution/src/VerbsToken.sol

184: function burn(uint256 verbId) public override onlyMinter nonReentrant {
185:        _burn(verbId);
186:        emit VerbBurned(verbId);
187:    }

```

[L184-L187](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L184C4-L187C6)

### `SAVES ~24700 GAS` when every time this function called

```solidity
File : main/packages/revolution/src/VerbsToken.sol

209:   function setMinter(address _minter) external override onlyOwner nonReentrant whenMinterNotLocked {
210:        require(_minter != address(0), "Minter cannot be zero address");
211:        minter = _minter;
212:
213:        emit MinterUpdated(_minter);
214:     }
```

[L209-L214](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L209C1-L215C1)

### `SAVES ~24700 GAS` when every time this function called

```solidity
File : main/packages/revolution/src/VerbsToken.sol

230:    function setDescriptor(
231:        IDescriptorMinimal _descriptor
232:    ) external override onlyOwner nonReentrant whenDescriptorNotLocked {
233:        descriptor = _descriptor;
234:
235:        emit DescriptorUpdated(_descriptor);
236:    }

```

[L230-L236](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L230C1-L236C6)

### `SAVES ~24700 GAS` when every time this function called

```solidity
File : main/packages/revolution/src/VerbsToken.sol

252:  function setCultureIndex(ICultureIndex _cultureIndex) external onlyOwner whenCultureIndexNotLocked nonReentrant {
253:        cultureIndex = _cultureIndex;
254:
255:        emit CultureIndexUpdated(_cultureIndex);
256:    }
```

[L252-L256](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L252C1-L256C6)

## [G-05] Remove `Vote` struct as value from `votes` mapping in place of that use only `uin256 weight` as value

### `SAVE: 1 SLOT per new voterAddress as key`

Since voterAddress is already used as key in internal mapping of `votes` so no need to add again that address by adding Vote struct as value in internal mapping. It can save 1 SLOT per new voterAddress as key.

```solidity
File : main/packages/revolution/src/CultureIndex.sol
68:   // The mapping of all votes for a piece
69:   mapping(uint256 => mapping(address => Vote)) public votes;
```

[L68-L69](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L68C1-L69C63)

```solidity
File : main/packages/revolution/src/interfaces/ICultureIndex.sol
131: struct Vote {
132:     address voterAddress;
133:     uint256 weight;
134:    }
```

[L131-L134](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/interfaces/ICultureIndex.sol#L131C5-L134C6)

## [G-06] State variables should be cached in stack variables rather than re-reading them from storage

### `SAVE: 200 GAS, 2 SLOAD`

### NOTE: Not in bot report

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### Cache function external call `_calculateVoteWeight` and `(quorumVotesBPS * newPiece.totalVotesSupply) / 10_000` to save 2 SLOAD(~200 GAS)

```diff
File : main/packages/revolution/src/CultureIndex.sol

209: function createPiece(
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
+        uint256 _totalCalculatedVoteWeight = _calculateVoteWeight(
+            erc20VotingToken.totalSupply(),
+            erc721VotingToken.totalSupply()
+        );

+        uint256 _quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;

        newPiece.pieceId = pieceId;
-226:    newPiece.totalVotesSupply = _calculateVoteWeight(
-227:          erc20VotingToken.totalSupply(),
-228:          erc721VotingToken.totalSupply()
-229:       );
+226:    newPiece.totalVotesSupply = _totalCalculatedVoteWeight;
        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
        newPiece.metadata = metadata;
        newPiece.sponsor = msg.sender;
        newPiece.creationBlock = block.number;
-       newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
+       newPiece.quorumVotes = _quorumVotes;

        for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
        }

-240:     emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
+240:     emit PieceCreated(pieceId, msg.sender, metadata, _quorumVotes, _totalCalculatedVoteWeight);

        // Emit an event for each creator
        for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }

-       return newPiece.pieceId;
+       return pieceId;
248:    }

```

## [G-07] State address variable which is fix for chain can be marked immutable and initialize in constructor

### Saves: 1 SLOT and 1 SLOAD every time it is being read in contract

```solidity
File : main/packages/revolution/src/AuctionHouse.sol

54:  address public WETH;

95: constructor(address _manager) payable initializer {
96:        manager = IRevolutionBuilder(_manager);
97:    }

113: function initialize(
        ...
143:        WETH = _weth;//@audit this can be fixed based on chain so can be immutable in implementation
    }

```

[L54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L54), [95-97](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L95C5-L97C6), [113-143](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L113C5-L144C6)

```diff
File : main/packages/revolution/src/AuctionHouse.sol

-   address public WETH;
+   address public immutable WETH;


-    constructor(address _manager) payable initializer {
-        manager = IRevolutionBuilder(_manager);
-    }
+    constructor(address _manager, address _weth) payable initializer {
+        manager = IRevolutionBuilder(_manager);
+        WETH = _weth;
+    }

    function initialize(
        address _erc721Token,
        address _erc20TokenEmitter,
        address _initialOwner,
-       address _weth,
        IRevolutionBuilder.AuctionParams calldata _auctionParams
    ) external initializer {
        require(msg.sender == address(manager), "Only manager can initialize");
        require(_weth != address(0), "WETH cannot be zero address");

        __Pausable_init();
        __ReentrancyGuard_init();
        __Ownable_init(_initialOwner);

        _pause();

        require(
            _auctionParams.creatorRateBps >= _auctionParams.minCreatorRateBps,
            "Creator rate must be greater than or equal to the creator rate"
        );

        verbs = IVerbsToken(_erc721Token);
        erc20TokenEmitter = IERC20TokenEmitter(_erc20TokenEmitter);
        timeBuffer = _auctionParams.timeBuffer;
        reservePrice = _auctionParams.reservePrice;
        minBidIncrementPercentage = _auctionParams.minBidIncrementPercentage;
        duration = _auctionParams.duration;
        creatorRateBps = _auctionParams.creatorRateBps;
        entropyRateBps = _auctionParams.entropyRateBps;
        minCreatorRateBps = _auctionParams.minCreatorRateBps;
-       WETH = _weth;
    }

```

## [G-08] Do not assign same struct value second time to same variable if it is already assigned it saves multiple MLOAD and MSTORE

```diff
File : main/packages/revolution/src/VerbsToken.sol

281:    function _mintTo(address to) internal returns (uint256) {
282:        ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

        // Check-Effects-Interactions Pattern
        // Perform all checks
        require(
            artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
            "Creator array must not be > MAX_NUM_CREATORS"
        );

        // Use try/catch to handle potential failure
-292:        try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
+292:        try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory /*_artPiece*/) {
-293:            artPiece = _artPiece;
            uint256 verbId = _currentVerbId++;

```

[L281-L294](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L281C1-L294C47)

## [G-09] Using `storage` instead of memory for structs/arrays saves gas (saves 4200 GAS)

### `SAVES: 4200 GAS`

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

### Using storage variables can potentially save gas by `eliminating the cost of copying the struct from storage to memory` :`SAVES 4200 GAS`

```diff

-172: IAuctionHouse.Auction memory _auction = auction;
+172: IAuctionHouse.Auction storage _auction = auction;

-337: IAuctionHouse.Auction memory _auction = auction;
+337: IAuctionHouse.Auction storage _auction = auction;

```

[L172](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L172), [L337](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L337)

## [G-10] Refactor code to fail early

### Refactor require statement that checks input argument should first to fail early

```diff
File : main/packages/revolution/src/AuctionHouse.sol

171: function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
+175:     require(bidder != address(0), "Bidder cannot be zero address");
172:    IAuctionHouse.Auction memory _auction = auction;
173:
174:    //require bidder is valid address
-175:     require(bidder != address(0), "Bidder cannot be zero address");

```

[L172-L175](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L171C5-L175C72)

### First check `voter` for address(0) and then check `pieceId` to `_currentPieceId` to save gas because `_currentPieceId` is a state variable so it is cheaper to check `voter` first

```diff
File : main/packages/revolution/src/CultureIndex.sol

307:    function _vote(uint256 pieceId, address voter) internal {
+309:        require(voter != address(0), "Invalid voter address");
308:        require(pieceId < _currentPieceId, "Invalid piece ID");
-309:        require(voter != address(0), "Invalid voter address");
310:        require(!pieces[pieceId].isDropped, "Piece has already been dropped");
311:        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");

```

[L307-L311](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L307C5-L311C87)

## [G-11] Refactor the `computeTotalReward` function to save gas

### We can refactor the return statement to save 3 Mul opcodes and 3 DIV opcodes which is going to save 15, 15 gas respectively.

```solidity
File : main/packages/protocol-rewards/src/abstract/RewardSplits.sol

40: function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
41:    if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
42:
43:        return
44:            (paymentAmountWei * BUILDER_REWARD_BPS) /
45:            10_000 +
46:            (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
47:            10_000 +
48:            (paymentAmountWei * DEPLOYER_REWARD_BPS) /
49:            10_000 +
50:            (paymentAmountWei * REVOLUTION_REWARD_BPS) /
51:            10_000;
52:    }

```

[L40-L52](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40C1-L52C6)

This code first calculates the total of all the Base Points (BPS) and then multiplies paymentAmountWei by the total BPS and finally divides the result by 10,000. This reduces the number of division operations to one, potentially optimizing gas usage in the contract execution.

```diff

    function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

-       return
-            (paymentAmountWei * BUILDER_REWARD_BPS) /
-            10_000 +
-            (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
-            10_000 +
-            (paymentAmountWei * DEPLOYER_REWARD_BPS) /
-            10_000 +
-            (paymentAmountWei * REVOLUTION_REWARD_BPS) /
-            10_000;
+       uint256 totalBps = BUILDER_REWARD_BPS +
+                   PURCHASE_REFERRAL_BPS +
+                   DEPLOYER_REWARD_BPS +
+                   REVOLUTION_REWARD_BPS;

+     uint256 totalReward = (paymentAmountWei * totalBps) / 10_000;

+      return totalReward;
    }

```

## [G-12] Refactor the `_vote` function to save gas

### Remove both `!` opcode because it will work same as with `!` opcode and cache `totalVoteWeights[pieceId]` first to save 1 SLOAD :`SAVES 2 ! opcode and 1 SLOAD(~100 GAS)`

```diff
File : main/packages/revolution/src/CultureIndex.sol

307:    function _vote(uint256 pieceId, address voter) internal {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        require(voter != address(0), "Invalid voter address");
        require(!pieces[pieceId].isDropped, "Piece has already been dropped");
-311:    require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
+311:    require((votes[pieceId][voter].voterAddress == address(0)), "Already voted");

        uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);
        require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");

        votes[pieceId][voter] = Vote(voter, weight);
+319:    uint256 totalWeight = totalVoteWeights[pieceId];

-317:        totalVoteWeights[pieceId] += weight;
+317:        totalVoteWeights[pieceId] = totalWeight + weight;

-319:    uint256 totalWeight = totalVoteWeights[pieceId];

        // TODO add security consideration here based on block created to prevent flash attacks on drops?
        maxHeap.updateValue(pieceId, totalWeight);
        emit VoteCast(pieceId, voter, weight, totalWeight);
324:    }

```

[L307-L324](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L307C1-L324C6)

## [G-13] Remove unnecessary checks

### Remove Second require check because `creatorRateBps` is also checked for `10000` in `setCreatorRateBps` function

```diff
File : main/packages/revolution/src/AuctionHouse.sol

233: function setMinCreatorRateBps(uint256 _minCreatorRateBps) external onlyOwner {
234:     require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
-235:     require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");

```

[L233-L235](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L233C5-L235C104)

### Remove first if block because below check insures that from is not address 0 and both should be equal to pass the below check

```diff
File : main/packages/revolution/src/CultureIndex.sol

-438: if (from == address(0)) revert ADDRESS_ZERO();
439:
440:  // Ensure signature is valid
441:  if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();


```

[L438-L441](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L438C7-L441C100)

## [G-14] Missing zero-address check in constructor

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

```solidity
File : main/packages/revolution/src/AuctionHouse.sol

95: constructor(address _manager) payable initializer {
96:        manager = IRevolutionBuilder(_manager);
97:    }
```

[95-97](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L95C5-L97C6)

```solidity
File : main/packages/revolution/src/AuctionHouse.sol

92:   constructor(address _manager) payable initializer {
93:      manager = IRevolutionBuilder(_manager);
94:   }

```

[92-94](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L92C1-L94C6)

```solidity
File : main/packages/revolution/src/ERC20TokenEmitter.sol

64: constructor(
65:        address _manager,
66:        address _protocolRewards,
67:        address _protocolFeeRecipient
68:    ) payable TokenEmitterRewards(_protocolRewards, _protocolFeeRecipient) initializer {
69:        manager = IRevolutionBuilder(_manager);
70:    }

```

[64-70](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L64C5-L70C6)

```solidity
File : main/packages/revolution/src/MaxHeap.sol

30: constructor(address _manager) payable initializer {
31:        manager = IRevolutionBuilder(_manager);
32:    }

```

[30-32](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L30C5-L32C6)

```solidity
File : main/packages/revolution/src/NontransferableERC20Votes.so

44: constructor(address _manager) payable initializer {
45:        manager = IRevolutionBuilder(_manager);
46:    }

```

[L44-L46](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L44C5-L46C6)

```solidity
File : revolution/src/VerbsToken.sol

116: constructor(address _manager) payable initializer {
117:        manager = IRevolutionBuilder(_manager);
118:  }
```

[L116-L118](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L116C1-L118C6)
