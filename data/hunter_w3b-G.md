# Gas-Optmization for Revolution Protocol

## [G-01] Avoid contract existence checks by using low level calls

**Issue Description**\
Prior to Solidity 0.8.10, the compiler would insert extra code to check for contract existence before external function calls,
even if the call had a return value. This wasted gas by performing a check that wasn't necessary.

**Proposed Optimization**\
For functions that make external calls with return values in Solidity <0.8.10, optimize the code to use low-level calls instead
of regular calls. Low-level calls skip the unnecessary contract existence check.

Example:

```solidity
//Before:

contract C {
  function f() external returns(uint) {
    address(otherContract).call(abi.encodeWithSignature("func()"));
  }
}


//After:

contract C {
  function f() external returns(uint) {
    (bool success,) = address(otherContract).call(abi.encodeWithSignature("func()"));
    require(success);
    return decodeReturnValue();
  }
}
```

**Estimated Gas Savings**\
Each avoided EXTCODESIZE check saves 100 gas. If 10 external calls are made in a common function, this would save 1000 gas
total.

**Attachments**

- **Code Snippets**

```solidity
File: revolution/src/AuctionHouse.sol

370                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;

371                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L370

```solidity
File: src/abstract/RewardSplits.sol

80        protocolRewards.depositRewards{ value: totalReward }(
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L80

```solidity
File: src/CultureIndex.sol

221        maxHeap.insert(pieceId, 0);

227            erc20VotingToken.totalSupply(),

228            erc721VotingToken.totalSupply()

230        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();

295                erc20VotingToken.getPastVotes(account, blockNumber),

296                erc721VotingToken.getPastVotes(account, blockNumber)
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L221

## [G-02] Using assembly to revert with an error message

**Issue Description**\
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.

**Estimated Gas Savings**\
Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042


contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734


contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message
with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks
the solidity compiler does under the hood.

**Attachments**

- **Code Snippets**

```solidity
File: src/MaxHeap.sol

42        require(msg.sender == admin, "Sender is not the admin");

56        require(msg.sender == address(manager), "Only manager can initialize");

79        require(pos != 0, "Position should not be zero");

157        require(size > 0, "Heap is empty");

170        require(size > 0, "Heap is empty");

183        if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L183

```solidity
File: src/CultureIndex.sol


117        require(msg.sender == address(manager), "Only manager can initialize");

119        require(_cultureIndexParams.quorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "invalid quorum bps");

120        require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");

121        require(_erc721VotingToken != address(0), "invalid erc721 voting token");

122        require(_erc20VotingToken != address(0), "invalid erc20 voting token");

160        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

182        require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

186            require(creatorArray[i].creator != address(0), "Invalid creator address");

190        require(totalBps == 10_000, "Total BPS must sum up to 10,000");


308        require(pieceId < _currentPieceId, "Invalid piece ID");

309        require(voter != address(0), "Invalid voter address");

310        require(!pieces[pieceId].isDropped, "Piece has already been dropped");

311        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");

313        uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);

314        require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");

377        if (!success) revert INVALID_SIGNATURE();

404            if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();

427        require(deadline >= block.timestamp, "Signature expired");

438        if (from == address(0)) revert ADDRESS_ZERO();

441        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

452        require(pieceId < _currentPieceId, "Invalid piece ID");

462        require(pieceId < _currentPieceId, "Invalid piece ID");

487        require(maxHeap.size() > 0, "Culture index is empty");

499        require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");

520        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");

523        require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");

545        if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L377

```solidity
File: src/NontransferableERC20Votes.sol

69        require(msg.sender == address(manager), "Only manager can initialize");

95        revert TRANSFER_NOT_ALLOWED();

102        revert TRANSFER_NOT_ALLOWED();

109        revert TRANSFER_NOT_ALLOWED();

116        revert TRANSFER_NOT_ALLOWED();

129            revert ERC20InvalidReceiver(address(0));

142        revert TRANSFER_NOT_ALLOWED();

149        revert TRANSFER_NOT_ALLOWED();

156        revert TRANSFER_NOT_ALLOWED();
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L95

```solidity
File:

91        require(msg.sender == address(manager), "Only manager can initialize");

96        require(_treasury != address(0), "Invalid treasury address");

158        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

160        require(msg.value > 0, "Must send ether");

162        require(addresses.length == basisPointSplits.length, "Parallel arrays required");

192        require(success, "Transfer failed.");

197            require(success, "Transfer failed.");

217        require(bpsSum == 10_000, "bps must add up to 10_000");

238        require(amount > 0, "Amount must be greater than 0");

255        require(etherAmount > 0, "Ether amount must be greater than 0");

272        require(paymentAmount > 0, "Payment amount must be greater than 0");

289        require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

300        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");

310        require(_creatorsAddress != address(0), "Invalid address");
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L91

```solidity
File: src/AuctionHouse.sol

120        require(msg.sender == address(manager), "Only manager can initialize");

121        require(_weth != address(0), "WETH cannot be zero address");

129        require(

175        require(bidder != address(0), "Bidder cannot be zero address");

176        require(_auction.verbId == verbId, "Verb not up for auction");

178        require(block.timestamp < _auction.endTime, "Auction expired");

179        require(msg.value >= reservePrice, "Must send at least reservePrice");

180        require(

218        require(

222        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");

234        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");

235        require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");

238        require(

254        require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

311        require(gasleft() >= MIN_TOKEN_MINT_GAS_THRESHOLD, "Insufficient gas for creating auction");

339        require(_auction.startTime != 0, "Auction hasn't begun");

340        require(!_auction.settled, "Auction has already been settled");

341        require(block.timestamp >= _auction.endTime, "Auction hasn't completed");

421        if (address(this).balance < _amount) revert("Insufficient balance");

441            if (!wethSuccess) revert("WETH transfer failed");

454        if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L421

```solidity
File: src/VerbsToken.sol

76        require(!isMinterLocked, "Minter is locked");

84        require(!isCultureIndexLocked, "CultureIndex is locked");

92        require(!isDescriptorLocked, "Descriptor is locked");

100        require(msg.sender == minter, "Sender is not the minter");

137        require(msg.sender == address(manager), "Only manager can initialize");

139        require(_minter != address(0), "Minter cannot be zero address");

140        require(_initialOwner != address(0), "Initial owner cannot be zero address");

210        require(_minter != address(0), "Minter cannot be zero address");

274        require(verbId <= _currentVerbId, "Invalid piece ID");

286        require(

317            revert("dropTopVotedPiece failed");

330        require(manager.isRegisteredUpgrade(_getImplementation(), _newImpl), "Invalid upgrade");
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L76

## [G-03] Use assembly to reuse memory space when making more than one external call

**Issue Description**\
When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not
clear or reuse memory, so it expands memory each time.

**Proposed Optimization**\
Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments
without expanding memory further.

**Estimated Gas Savings**\
Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000
gas. With larger arguments or return data, the savings would be even greater.

```solidity
See the example below;

contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also
reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding
memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it
wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas
by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity
overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within
that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00
before exiting the assembly block.

**Attachments**

- **Code Snippets**

```solidity
File: src/AuctionHouse.sol

355            verbs.burn(_auction.verbId);
359                verbs.burn(_auction.verbId);
361            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);
370                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
371                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;
385                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L359

```solidity
File: src/CultureIndex.sol

227            erc20VotingToken.totalSupply(),
228            erc721VotingToken.totalSupply()
230        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();


295                erc20VotingToken.getPastVotes(account, blockNumber),
296                erc721VotingToken.getPastVotes(account, blockNumber)
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L227-L228

This report discusses how using inline assembly can optimize gas costs when making multiple external calls by reusing
memory space, rather than expanding memory separately for each call. This can save thousands of gas compared to the
solidity compiler's default behavior.

## [G-04] Don't make variables public unless necessary

**Issue Description**\
Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates
a public getter function of the same name, increasing the contract size.

**Proposed Optimization**\
Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private
or internal visibility.

**Estimated Gas Savings**\
The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up
over many transactions targeting a contract with public state variables that don't need to be public.

**Attachments**

- **Code Snippets**

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

```solidity
File: src/libs/VRGDAC.sol

16    int256 public immutable targetPrice;

18    int256 public immutable perTimeUnit;

20    int256 public immutable decayConstant;

22    int256 public immutable priceDecayPercent;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L16-L22

## [G-05] It is sometimes cheaper to cache calldata

**Issue Description**\
While the calldataload opcode is relatively cheap, directly using it in a loop or multiple times can still result in unnecessary
bytecode. Caching the loaded calldata first may allow for optimization opportunities.

**Proposed Optimization**\
Cache calldata values in a local variable after first load, then reference the local variable instead of repeatedly using
calldataload.

**Estimated Gas Savings**\
Exact savings vary depending on contract, but caching calldata parameters can save 5-20 gas per usage by avoiding extra calldataload
opcodes. Larger functions with many parameter uses see more benefit.

**Attachments**

- **Code Snippets**

```solidity
File: src/CultureIndex.sol

159    function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {

179    function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {

210        ArtPieceMetadata calldata metadata,

211        CreatorBps[] calldata creatorArray

342    function voteForMany(uint256[] calldata pieceIds) public nonReentrant {

353    function _voteForMany(uint256[] calldata pieceIds, address from) internal {

369        uint256[] calldata pieceIds,

391        uint256[][] calldata pieceIds,

421        uint256[] calldata pieceIds,

```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L159

```solidity
File: src/NontransferableERC20Votes.sol

54        string calldata _name,

55        string calldata _symbol

67        IRevolutionBuilder.ERC20TokenParams calldata _erc20TokenParams
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L54

```solidity
File: src/ERC20TokenEmitter.sol

153        address[] calldata addresses,

154        uint[] calldata basisPointSplits,

155        ProtocolRewardAddresses calldata protocolRewardsRecipients
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L153

## [G-06] Shorten arrays with inline assembly

**Issue Description**\
When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating
storage.

**Proposed Optimization**\
Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new
array.

**Estimated Gas Savings**\
Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending
on original length.

```solidity
function shorten(uint[] storage array, uint newLen) internal {

  assembly {
    sstore(array_slot, newLen)
  }

}

// Rather than:
function shorten(uint[] storage array, uint newLen) internal {

  uint[] memory newArray = new uint[](newLen);

  for(uint i = 0; i < newLen; i++) {
    newArray[i] = array[i];
  }

  delete array;
  array = newArray;

}
```

Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas
savings.

**Attachments**

- **Code Snippets**

```solidity
File: src/AuctionHouse.sol

374                uint256[] memory vrgdaSplits = new uint256[](numCreators);

375                address[] memory vrgdaReceivers = new address[](numCreators);
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L374

## [G-07] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

**Issue Description**\
The abi.encodePacked() function is used to concatenate multiple arguments into a byte array prior to 0.8.4. However, since
0.8.4 the bytes.concat() function was introduced which performs the same role but is preferred since it avoids ABI encoding
overhead.

**Proposed Optimization**\
Replace uses of abi.encodePacked() with bytes.concat() where multiple arguments need to be concatenated into a byte array.

**Estimated Gas Savings**\
Using bytes.concat() instead of abi.encodePacked() saves approximately 300-500 gas per concatenation by avoiding ABI encoding.

**Attachments**

- **Code Snippets**

```solidity
File: src/VerbsToken.sol

162        return string(abi.encodePacked("ipfs://", _contractURIHash));
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L162

## [G-08] Consider using ERC721A instead of ERC721

[ERC721A](https://www.erc721a.org/) is an improved implementation of IERC721 with significant gas savings for minting
multiple NFTs in a single transaction.

```solidity
File: src/VerbsToken.sol

22  import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L22

## [G-09] Expressions for constatn values such as a call to KECCAK256(), should use immutable rather than constant

**Issue Description**\
The variable VOTE_TYPEHASH is declared as constant but its value is computed from a call to keccak256(). Since this value does not change, it should be declared as immutable to properly indicate that its value is computed once at contract deployment rather than being a true constant.

**Proposed Optimization**\
Change the declaration of VOTE_TYPEHASH from constant to immutable:

```solidity
immutable bytes32 public VOTE_TYPEHASH = keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");
```

**Estimated Gas Savings**\
Changing constant to immutable helps properly indicate that the value of VOTE_TYPEHASH is computed at contract deployment rather than being a true constant. This improves clarity without impacting gas costs.

**Attachments**

- **Code Snippets**

```solidity
File: src/CultureIndex.sol

29    bytes32 public constant VOTE_TYPEHASH =
30        keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L29-L30

## [G-10] Use nested if and, avoid multiple check combinations &&

**Issue Description**\
The code snippets are checking multiple conditions using && operators. This can be optimized by replacing with nested if statements to avoid unnecessary checks.

**Proposed Optimization**\

1. For MaxHeap.sol L102:

   ```solidity
   if (pos >= (size / 2)) {
   if (pos <= size) return;
   }
   ```

2. For ERC20TokenEmitter.sol L201:

   ```solidity
   if (totalTokensForCreators > 0) {
   if (creatorsAddress != address(0)) {
       // code
   }
   }
   ```

3. For AuctionHouse.sol L383:

   ```solidity
   if (creatorsShare > 0) {
   if (entropyRateBps > 0) {
   // code
   }
   }
   ```

**Estimated Gas Savings**\
Replacing && with nested ifs avoids evaluating the second condition unnecessarily if the first fails, which can provide marginal gas savings.

**Attachments**

- **Code Snippets**

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

```solidity
File: src/AuctionHouse.sol

383                if (creatorsShare > 0 && entropyRateBps > 0) {
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L383

## [G-11] Use selfbalance() instead of address(this).balance for contract balance checks

**Issue Description**\
The code is directly using address(this).balance to check the contract's ether balance, which can be inefficient in some cases.

**Proposed Optimization**\
Replace address(this).balance with selfbalance() where possible. The selfbalance() function from the Yul layer provides a direct reference to the contract's balance without having to perform an external call.

**Estimated Gas Savings**\
selfbalance() can save 200-400 gas compared to address(this).balance depending on the scenario. The savings comes from avoiding the external call.

**Attachments**

- **Code Snippets**

```solidity
File: src/AuctionHouse.sol

348        if (address(this).balance < reservePrice) {

421        if (address(this).balance < _amount) revert("Insufficient balance");
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L348

This optimization may provide marginal gas savings by leveraging the more direct selfbalance() reference where the contract just needs to check its own balance without an external call. It's worth benchmarking both versions to confirm if any savings are realized in the specific contract/function.

## [G-12] Use pre-increment and pre-decrement instead of +1/-1

**Issue Description**\
The code is using +1 and -1 to increment and decrement variables.

**Proposed Optimization**\
Replace instances of +1 with ++ for pre-decrement, and -1 with -- for pre-increment.

**Estimated Gas Savings**\
pre-increment/decrement can save 2 gas per operation compared to adding/subtracting 1.

**Attachments**

- **Code Snippets**

```solidity
File: src/MaxHeap.sol

80        return (pos - 1) / 2;//@audit >>> - 1

95        uint256 left = 2 * pos + 1; //@audit >>> + 1
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L80

## [G-13] Avoid using nonReentrant where onlyOwner

**Issue Description**\
Some functions are using both the onlyOwner and nonReentrant modifiers unnecessarily.The onlyOwner modifier already provides the necessary access control and protection against reentrancy by the owner.The onlyOwner modifier alone is sufficient to restrict a function to the owner and trust that the owner will not initiate reentrancy.

**Proposed Optimization**\
Remove nonReentrant from functions already protected by onlyOwner. The onlyOwner modifier alone provides sufficient protection against reentrancy without any gas costs.

**Attachments**

- **Code Snippets**

```solidity
File: src/ERC20TokenEmitter.sol

309    function setCreatorsAddress(address _creatorsAddress) external override onlyOwner nonReentrant {
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L309

```solidity
File: src/VerbsToken.sol

177    function mint() public override onlyMinter nonReentrant returns (uint256) {

184    function burn(uint256 verbId) public override onlyMinter nonReentrant {

209    function setMinter(address _minter) external override onlyOwner nonReentrant whenMinterNotLocked {

232    ) external override onlyOwner nonReentrant whenDescriptorNotLocked {

242    function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L177

## [G-14] Use "" instead of new bytes(0)

**Issue Description**\
Some code is initializing empty byte arrays using new bytes(0) for empty calldata.

**Proposed Optimization**\
Replace new bytes(0) with the empty string "" for same semantic but lower gas cost.

For_Example:

```solidity
// Before
(bool success, ) = to.call{value: value, gas: 30_000}(new bytes(0));

// After
(bool success, ) = to.call{value: value, gas: 30_000}("");
```

**Estimated Gas Savings**\
costs ~200 less gas than new bytes(0).

**Attachments**

- **Code Snippets**

```solidity
File: src/ERC20TokenEmitter.sol

191        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));

196            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L191

Using "" instead of new bytes(0) provides the same empty byte array value but avoids constructing a new object, providing a minor gas savings for equivalent functionality.

## [G-15] Expensive operation inside a for loop

**Issue Description**\
The `_settleAuction()` function makes external `_safeTransferETHWithFallback()` calls inside a for loop to distribute auction proceeds to creators.

**Proposed Optimization**\
Move expensive external calls out of the loop by:

- Computing splits & recipients arrays first
- Making a single bulk ETH transfer to the ERC20 contract
- Having ERC20 contract distribute tokens to recipients

**Estimated Gas Savings**\
Reduces external calls from O(n) to O(1). Can save >10k gas for larger creator lists.

**Attachments**

- **Code Snippets**

```solidity
File: src/AuctionHouse.sol


384                    for (uint256 i = 0; i < numCreators; i++) {
385                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
386                        vrgdaReceivers[i] = creator.creator;
387                        vrgdaSplits[i] = creator.bps;
388
389                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
390                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
391                        ethPaidToCreators += paymentAmount;
392
393                        //Transfer creator's share to the creator
394                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
395                    }
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L384-L395

## [G-16] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are Solmate and Solady.

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.
