# Gas Optimization

## [G-01] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

```solidity
File: src/VerbsToken.sol

162        return string(abi.encodePacked("ipfs://", _contractURIHash));
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L162

## [G-02] Use nested if and, avoid multiple check combinations &&

```solidity
File: src/AuctionHouse.sol

383                if (creatorsShare > 0 && entropyRateBps > 0) {
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L383

```solidity
File: src/MaxHeap.sol

102        if (pos >= (size / 2) && pos <= size) return;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L102

```solidity
File: src/ERC20TokenEmitter.sol

201         if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {

```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L201

## [G-03] Don't make variables public unless necessary

```solidity
File: src/ERC20TokenEmitter.sol

25    address public treasury;

28    NontransferableERC20Votes public token;

31    VRGDAC public vrgdac;

34    uint256 public startTime;

39    int256 public emittedTokenWad;

42    uint256 public creatorRateBps;

45    uint256 public entropyRateBps;

48    address public creatorsAddress;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L25

```solidity
File: src/MaxHeap.sol

16    address public admin;

65    mapping(uint256 => uint256) public heap;

67    uint256 public size = 0;

70    mapping(uint256 => uint256) public valueMapping;

73    mapping(uint256 => uint256) public positionMapping;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L16

```solidity
File: src/CultureIndex.sol

29    bytes32 public constant VOTE_TYPEHASH =

33    mapping(address => uint256) public nonces;

36    MaxHeap public maxHeap;

39    ERC20VotesUpgradeable public erc20VotingToken;

42    ERC721CheckpointableUpgradeable public erc721VotingToken;

45    uint256 public erc721VotingTokenWeight;

48    uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%

51    uint256 public minVoteWeight;

54    uint256 public quorumVotesBPS;

57    string public name;

60    string public description;

63    mapping(uint256 => ArtPiece) public pieces;

66    uint256 public _currentPieceId;

69    mapping(uint256 => mapping(address => Vote)) public votes;

72    mapping(uint256 => uint256) public totalVoteWeights;

75    uint256 public constant MAX_NUM_CREATORS = 100;

78    address public dropperAdmin;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L29

```solidity
File: src/libs/VRGDAC.sol

16    int256 public immutable targetPrice;

18    int256 public immutable perTimeUnit;

20    int256 public immutable decayConstant;

22    int256 public immutable priceDecayPercent;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L16-L22

```solidity
File: src/AuctionHouse.sol

48    IVerbsToken public verbs;

51    IERC20TokenEmitter public erc20TokenEmitter;

54    address public WETH;

57    uint256 public timeBuffer;

60    uint256 public reservePrice;

63    uint8 public minBidIncrementPercentage;

66    uint256 public creatorRateBps;

69    uint256 public minCreatorRateBps;

72    uint256 public entropyRateBps;

75    uint256 public duration;

78    IAuctionHouse.Auction public auction;

85    IRevolutionBuilder public immutable manager;

88    uint32 public constant MIN_TOKEN_MINT_GAS_THRESHOLD = 750_000;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L48

```solidity
File: src/VerbsToken.sol

42    address public minter;

45    IDescriptorMinimal public descriptor;

48    ICultureIndex public cultureIndex;

51    bool public isMinterLocked;

54    bool public isCultureIndexLocked;

57    bool public isDescriptorLocked;

66    mapping(uint256 => ICultureIndex.ArtPiece) public artPieces;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L42

## [G-04] Expression `` is cheaper than new bytes(0)

```solidity
File: src/ERC20TokenEmitter.sol

191        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));

196            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L191

## [G-05] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File: src/AuctionHouse.sol

374                uint256[] memory vrgdaSplits = new uint256[](numCreators);

375                address[] memory vrgdaReceivers = new address[](numCreators);
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L374

## [G-06] Expressions for constatn values such as a call to KECCAK256(), should use immutable rather than constant

```solidity
File: src/CultureIndex.sol

29    bytes32 public constant VOTE_TYPEHASH =
30        keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L29-L30

## [G-07] Consider using ERC721A instead of ERC721

[ERC721A](https://www.erc721a.org/) is an improved implementation of IERC721 with significant gas savings for minting
multiple NFTs in a single transaction.

```solidity
File: src/VerbsToken.sol

22  import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L22

## [G-08] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are Solmate and Solady.

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.