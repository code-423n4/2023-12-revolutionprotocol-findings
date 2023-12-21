# Revolution

### Low Risk Issues List

| Number | Issues Details | Context 
| --- | --- | --- |
| [L-01] | Hard-coding the maxGasLimit variable may cause problems in the future | 1 |
| [L-02] | Owner can Brick/Lock contract | 2 |

Total 2 issues

### Non-Critical Issues List

| Number | Issues Details | Context |
| --- | --- | --- |
| [N-01] | Floating pragma | 22 |
| [N-02] | Use of different pragma versions | 3 |
| [N-03] | NatSpec comments should be increased in contracts | All contracts |
| [N-04] | Function writing that does not comply with the Solidity Style Guide | All contracts |
| [N-05] | Lack of event emission after critical initialize() function | 4 |
| [N-06] | Add parameter to Event-Emit | 2 |
| [N-07] | NatSpec is missing | 2 |
| [N-08] | Open Todos | 3 |

Total 8 issues

### [L-01] Hard-coding the `maxGasLimit` variable may cause problems in the future

The variable `maxGasLimit` is defined as `constant` and its value is hardcoded and cannot be changed later

EVM-Based blockchains are hardforked and there is no such 
thing as Gas Limit etc. values may change, this has happened in the 
past, so it is recommended to have this value updated in the future

```solidity
1 results - 1 file

packages/revolution/src/AuctionHouse.sol:
88: uint32 public constant MIN_TOKEN_MINT_GAS_THRESHOLD = 750_000;
```

### [L-02] Owner has permission to lock minter and descriptor, once locked, the action can’t be undone

When the owner decides to lock the minter and descriptor these actions can’t be undone and will brick the contract. It is recommended to add a function to set these values back to false.

```solidity
2 results - 1 file

packages/revolution/src/VerbsToken.sol:
/**
     * @notice Lock the minter.
     * @dev This cannot be reversed and is only callable by the owner when not locked.
     */
    function lockMinter() external override onlyOwner whenMinterNotLocked {
        isMinterLocked = true;

        emit MinterLocked();
    }

/**
     * @notice Lock the descriptor.
     * @dev This cannot be reversed and is only callable by the owner when not locked.
     */
    function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {
        isDescriptorLocked = true;

        emit DescriptorLocked();
    }
```

### [N-01] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags 
that they have been tested with thoroughly. Locking the pragma helps to 
ensure that contracts do not accidentally get deployed using, for 
example, an outdated compiler version that might introduce bugs that 
affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List:

```
22 files

packages/revolution/src/AuctionHouse.sol:
18: pragma solidity ^0.8.22;

packages/revolution/src/NontransferableERC20Votes.sol:
4: pragma solidity ^0.8.22;

packages/revolution/src/MaxHeap.sol:
2: pragma solidity ^0.8.22;

packages/revolution/src/ERC20TokenEmitter.sol:
2: pragma solidity ^0.8.22;

packages/revolution/src/Descriptor.sol:
18: pragma solidity ^0.8.22;

packages/revolution/src/CultureIndex.sol:
2: pragma solidity ^0.8.22;

packages/revolution/src/interfaces/IAuctionHouse.sol
18: pragma solidity ^0.8.22;

packages/revolution/src/interfaces/ICultureIndexEvents.sol
3: pragma solidity ^0.8.22;

packages/revolution/src/interfaces/IDescriptor.sol
18: pragma solidity ^0.8.22;

packages/revolution/src/interfaces/IDescriptorMinimal.sol
18: pragma solidity ^0.8.22;

packages/revolution/src/interfaces/IERC20TokenEmitter.sol
2: pragma solidity ^0.8.22;

packages/revolution/src/interfaces/INontransferableERC20Votes.sol
2: pragma solidity ^0.8.22;

packages/revolution/src/interfaces/IVerbsToken.sol
18: pragma solidity ^0.8.22;

packages/revolution/src/interfaces/IWETH.sol
3: pragma solidity ^0.8.22;

packages/revolution/src/governance/VerbsDAOLogicV1.sol
53: pragma solidity ^0.8.22;

packages/revolution/src/governance/DAOExecutor.sol
31: pragma solidity ^0.8.22;

packages/revolution/src/base/VotesUpgradeable.sol
3: pragma solidity ^0.8.20;

packages/revolution/src/base/ERC721Upgradeable.sol
4: pragma solidity ^0.8.22;

packages/revolution/src/base/ERC721EnumerableUpgradeable.sol
4: pragma solidity ^0.8.22;

packages/revolution/src/base/ERC721CheckpointableUpgradeable.sol
4: pragma solidity ^0.8.22;

packages/revolution/src/base/ERC20VotesUpgradeable.sol
4: pragma solidity ^0.8.20;

packages/revolution/src/base/ERC20Upgradeable.sol
4: pragma solidity ^0.8.20;
```

**Recommendation:**

Lock the pragma version and also consider known bugs (

https://github.com/ethereum/solidity/releases

) for the compiler version that is chosen.

### [N-02] Use of different pragma versions

Contracts should be deployed with the same compiler version and flags 
that they have been tested with thoroughly. Locking the pragma helps to 
ensure that contracts do not accidentally get deployed using, for 
example, an outdated compiler version that might introduce bugs that 
affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

```solidity
3 files

packages/revolution/src/base/VotesUpgradeable.sol
3: pragma solidity ^0.8.20;

packages/revolution/src/base/ERC20VotesUpgradeable.sol
4: pragma solidity ^0.8.20;

packages/revolution/src/base/ERC20Upgradeable.sol
4: pragma solidity ^0.8.20;
```

**Recommendation:**

Lock the pragma version and also consider known bugs (

https://github.com/ethereum/solidity/releases

) for the compiler version that is chosen.

### [N-03] Natspec is missing

Check the Solidity NatSpec format for all contracts

https://docs.soliditylang.org/en/latest/natspec-format.html 

### [N-04]**`unction writing` that does not comply with the `Solidity Style Guide`**

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they
 can call and to find the constructor and fallback definitions easier.
But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.23/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N-05] **Lack of event emission after critical `initialize()` functions**

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the `initialize()` functions:

```solidity
4 files

packages/revolution/src/AuctionHouse.sol
function initialize(
        address _erc721Token,
        address _erc20TokenEmitter,
        address _initialOwner,
        address _weth,
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
        WETH = _weth;
    }

packages/revolution/src/Descriptor.sol
function initialize(address _initialOwner, string calldata _tokenNamePrefix) external initializer {
        require(msg.sender == address(manager), "Only manager can initialize");

        // Ensure the caller is the contract manager
        require(msg.sender == address(manager), "Only manager can initialize");

        require(_initialOwner != address(0), "Initial owner cannot be zero address");

        // Setup ownable
        __Ownable_init(_initialOwner);

        tokenNamePrefix = _tokenNamePrefix;

        isDataURIEnabled = true;
    }

packages/revolution/src/ERC20TokenEmitter.sol
function initialize(
        address _initialOwner,
        address _erc20Token,
        address _treasury,
        address _vrgdac,
        address _creatorsAddress
    ) external initializer {
        require(msg.sender == address(manager), "Only manager can initialize");

        __Pausable_init();
        __ReentrancyGuard_init();

        require(_treasury != address(0), "Invalid treasury address");

        // Set up ownable
        __Ownable_init(_initialOwner);

        treasury = _treasury;
        creatorsAddress = _creatorsAddress;
        vrgdac = VRGDAC(_vrgdac);
        token = NontransferableERC20Votes(_erc20Token);
        startTime = block.timestamp;
    }

packages/revolution/src/governance/DAOExecutor.sol
function initialize(address _admin, uint256 _timelockDelay) external initializer {
        require(_timelockDelay >= MINIMUM_DELAY, "DAOExecutor::constructor: Delay must exceed minimum delay.");
        require(_timelockDelay <= MAXIMUM_DELAY, "DAOExecutor::setDelay: Delay must not exceed maximum delay.");

        require(msg.sender == address(manager), "Only manager can initialize");

        // Ensure a governor address was provided
        require(_admin != address(0), "DAOExecutor::initialize: Governor cannot be zero address");

        admin = _admin;
        delay = _timelockDelay;
    }
```

### [N-06] Add parameter to Event-Emit

Some event-emit description don’t have parameters. Add parameters for front-end website or client app, so they can broadcast events on the blockchain

Events with no value

```solidity
packages/revolution/src/VerbsToken.sol:

function lockMinter() external override onlyOwner whenMinterNotLocked {
        isMinterLocked = true;

        emit MinterLocked();
    }

function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {
        isDescriptorLocked = true;

        emit DescriptorLocked();
    }
```

### [N-07] NatSpec is missing

There are contracts where NatSpec is missing on functions. 

https://docs.soliditylang.org/en/latest/natspec-format.html 

This makes auditing code a lot harder

```solidity
packages/revolution/src/lib/SignedWadMath.sol

packages/revolution/src/lib/VRGDAC.sol
```

### [N-08] Open TODOs

**Context:**

```solidity
3 results - 3 files

packages/revolution/src/AuctionHouse.sol:
87: // TODO investigate this - The minimum gas threshold for creating an auction (minting VerbsToken)
88: uint32 public constant MIN_TOKEN_MINT_GAS_THRESHOLD = 750_000;

packages/revolution/src/CultureIndex.sol:
322: // TODO add security consideration here based on block created to prevent flash attacks on drops?
333: maxHeap.updateValue(pieceId, totalWeight);

packages/revolution/src/governance/VerbsDAOLogicV1.sol:
216: // TODO set these dynamic quorum params
217: // _setDynamicQuorumParams(
218: //     govParams_.dynamicQuorumParams.minQuorumVotesBPS,
219: //     govParams_.dynamicQuorumParams.maxQuorumVotesBPS,
220: //     govParams_.dynamicQuorumParams.quorumCoefficient
221: // );
```

**Recommendation:**

Use temporary TODOs as you work on a feature, but make sure to treat 
them before merging. Either add a link to a proper issue in your TODO, 
or remove it from the code.