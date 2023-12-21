## Gas Optimization

## Sumary

|No|Issu|instansce
|--|---|---------|
|[G-01]|Change public function visibility to external|30
|[G-02]|Unnecessary look up in if condition|5
|[G-03]|Using > 0 costs more gas than != 0 when used on a uint in a require() statement|12
|[G-04]|abi.encode() is less efficient than abi.encodePacked()|1
|[G-05]|Modifiers are redundant if used only once or not used at all. |1
|[G-06]|Use function instead of modifiers|2
|[G-07]|+= costs more gas than = + for state variables|5
|[G-08]|Use selfbalance() instead of address(this).balance|2
|[G-09]|Use ERC721A instead ERC721|6
|[G-10]|Use assembly for loops|8

### [G-01] Change public function visibility to external
Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity

file: packages/revolution/src/MaxHeap.sol

55   function initialize(address _initialOwner, address _admin) public initializer 

119   function insert(uint256 itemId, uint256 value) public onlyAdmin 

136   function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin

169  function getMax() public view returns (uint256, uint256) 

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L55
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L119
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L136
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L169

```solidity

file: packages/revolution/src/CultureIndex.sol

209  function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256)

332   function vote(uint256 pieceId) public nonReentrant {
        _vote(pieceId, msg.sender);
    }

342  function voteForMany(uint256[] calldata pieceIds) public nonReentrant {
        _voteForMany(pieceIds, msg.sender);
    }    

451   function getPieceById(uint256 pieceId) public view returns (ArtPiece memory) {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        return pieces[pieceId];
    }       

461    function getVote(uint256 pieceId, address voter) public view returns (Vote memory) {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        return votes[pieceId][voter];
    }    
470   function getTopVotedPiece() public view returns (ArtPiece memory) {
        return pieces[topVotedPieceId()];
    }    
486     function topVotedPieceId() public view returns (uint256) {
        require(maxHeap.size() > 0, "Culture index is empty");
        //slither-disable-next-line unused-return
        (uint256 pieceId, ) = maxHeap.getMax();
        return pieceId;
    }    
509    function quorumVotes() public view returns (uint256) {
        return
            (quorumVotesBPS * _calculateVoteWeight(erc20VotingToken.totalSupply(), erc721VotingToken.totalSupply())) /
            10_000;
    }    

519    function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");    
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L209
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L332
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L342
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L451
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L461
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L470
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L486
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L509
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L519

```solidity

file: packages/revolution/src/NontransferableERC20Votes.sol

87    function decimals() public view virtual override returns (uint8) {
        return 18;
    }

94    function transfer(address, uint256) public virtual override returns (bool) {
        revert TRANSFER_NOT_ALLOWED();
    }

108  function transferFrom(address, address, uint256) public virtual override returns (bool) {
        revert TRANSFER_NOT_ALLOWED();
    }    

115    function approve(address, uint256) public virtual override returns (bool) {
        revert TRANSFER_NOT_ALLOWED();
    }    

134   function mint(address account, uint256 amount) public onlyOwner {
        _mint(account, amount);
    }    
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L87
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L94
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L108
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L115
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L134

```solidity

file: packages/revolution/src/ERC20TokenEmitter.sol

112  function totalSupply() public view returns (uint) {
        // returns total supply issued so far
        return token.totalSupply();
    }

    function decimals() public view returns (uint8) {
        // returns decimals
        return token.decimals();
    }

122    function balanceOf(address _owner) public view returns (uint) {
        // returns balance of address
        return token.balanceOf(_owner);
    }
152   function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad)
237    function buyTokenQuote(uint256 amount) public view returns (int spentY)   

254   function getTokenQuoteForEther(uint256 etherAmount) public view returns (int gainedX) {
        require(etherAmount > 0, "Ether amount must be greater than 0");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L112-L122
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L152
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L237
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L254

```solidity

file: packages/revolution/src/VerbsToken.sol

161   function contractURI() public view returns (string memory) {
        return string(abi.encodePacked("ipfs://", _contractURIHash));
    }
177  function mint() public override onlyMinter nonReentrant returns (uint256) {
        return _mintTo(minter);
    }   
184  function burn(uint256 verbId) public override onlyMinter nonReentrant {
        _burn(verbId);
        emit VerbBurned(verbId);
    } 
193   function tokenURI(uint256 tokenId) public view override returns (string memory) {
        return descriptor.tokenURI(tokenId, artPieces[tokenId].metadata);
    }

    /**
     * @notice Similar to `tokenURI`, but always serves a base64 encoded data URI
     * with the JSON contents directly inlined.
     */
201    function dataURI(uint256 tokenId) public view override returns (string memory) {
        return descriptor.dataURI(tokenId, artPieces[tokenId].metadata);
    }     
273   function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
        require(verbId <= _currentVerbId, "Invalid piece ID");
        return artPieces[verbId];
    }      
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L161
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L177
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L184
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L193-L201
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L273

```solidity

file: packages/revolution/src/libs/VRGDAC.sol

47   function xToY(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {
        unchecked {
            return pIntegral(timeSinceStart, sold + amount) - pIntegral(timeSinceStart, sold);
        }
    }

    // given amount to pay and amount sold so far, returns # of tokens to sell - raw form
54    function yToX(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L47-L54

```solidity

file: abstract/RewardSplits.sol

40    function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {

54   function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
        return     
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L54

### [G-02] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: MaxHeap.sol

104  if (posValue < leftValue || posValue < rightValue)
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L104

```solidity

file: CultureIndex.sol

441   if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L441

```solidity

file: AuctionHouse.sol

268   if (auction.startTime == 0 || auction.settled)
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L268

```solidity

file: abstract/RewardSplits.sol

30  if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");

41     if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L30
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41

### [G‑03] Using > 0 costs more gas than != 0 when used on a uint in a require() statement
This change saves 6 gas per instance. The optimization works until solidity version 0.8.13 where there is a regression in gas costs.

```solidity

file: MaxHeap.sol

157   require(size > 0, "Heap is empty");

170   require(size > 0, "Heap is empty");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L157
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L170

```solidity

file: CultureIndex.sol

120   require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");

160     require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

163   require(bytes(metadata.image).length > 0, "Image URL must be provided");

165  require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");

167   require(bytes(metadata.text).length > 0, "Text must be provided");

487   require(maxHeap.size() > 0, "Culture index is empty");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L120
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L160
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L163
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L165
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L167
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L487

```solidity

file: ERC20TokenEmitter.sol

160   require(msg.value > 0, "Must send ether");

238    require(amount > 0, "Amount must be greater than 0");

255  require(etherAmount > 0, "Ether amount must be greater than 0");

272  require(paymentAmount > 0, "Payment amount must be greater than 0");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L160
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L238
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L255
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L272

### [G-04] abi.encode() is less efficient than abi.encodePacked()
Consider changing it if possible.

```solidity

file: CultureIndex.sol

431   voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L431

### [G-05] Modifiers are redundant if used only once or not used at all. 

```solidity

file: VerbsToken.sol

75   modifier whenMinterNotLocked() {
        require(!isMinterLocked, "Minter is locked");
        _;
    }

    /**
     * @notice Require that the CultureIndex has not been locked.
     */
    modifier whenCultureIndexNotLocked() {
        require(!isCultureIndexLocked, "CultureIndex is locked");
        _;
    }

    /**
     * @notice Require that the descriptor has not been locked.
     */
    modifier whenDescriptorNotLocked() {
        require(!isDescriptorLocked, "Descriptor is locked");
        _;
    }

    /**
     * @notice Require that the sender is the minter.
     */
99    modifier onlyMinter()

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L75-L99 

### [G-06] Use function instead of modifiers

```solidity

file: MaxHeap.sol

41  modifier onlyAdmin()
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L41

```solidity

file: VerbsToken.sol

75   modifier whenMinterNotLocked() {
        require(!isMinterLocked, "Minter is locked");
        _;
    }

    /**
     * @notice Require that the CultureIndex has not been locked.
     */
    modifier whenCultureIndexNotLocked() {
        require(!isCultureIndexLocked, "CultureIndex is locked");
        _;
    }

    /**
     * @notice Require that the descriptor has not been locked.
     */
    modifier whenDescriptorNotLocked() {
        require(!isDescriptorLocked, "Descriptor is locked");
        _;
    }

    /**
     * @notice Require that the sender is the minter.
     */
99    modifier onlyMinter()

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L75-L99 

### [G-07] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: CultureIndex.sol

187  totalBps += creatorArray[i].bps;

317 totalVoteWeights[pieceId] += weight;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L187
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L317

```solidity

file: ERC20TokenEmitter.sol

187 emittedTokenWad += totalTokensForBuyers;
188 if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

214  bpsSum += basisPointSplits[i];
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L187-L188
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L214


```solidity

file: AuctionHouse.sol

391   ethPaidToCreators += paymentAmount;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L391

### [G-08] Use selfbalance() instead of address(this).balance

```solidity

file: AuctionHouse.sol

348   if (address(this).balance < reservePrice)

421     if (address(this).balance < _amount) revert("Insufficient balance");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L348
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L421

###  [G-09] Use ERC721A instead ERC721
ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.

Reference: https://nextrope.com/erc721-vs-erc721a-2/

```solidity

file: CultureIndex.sol

16 import { ERC721CheckpointableUpgradeable } from "./base/ERC721CheckpointableUpgradeable.sol";s

42   ERC721CheckpointableUpgradeable public erc721VotingToken;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L16
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L42

```solidity

file: VerbsToken.sol

22 import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";

27 import { ERC721CheckpointableUpgradeable } from "./base/ERC721CheckpointableUpgradeable.sol";

39  ERC721CheckpointableUpgradeable

135   IRevolutionBuilder.ERC721TokenParams memory _erc721TokenParams
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L22
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L27
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L39
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L135

### [G-10] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

The final diffs have comments explaining the assembly code.

```solidity

file: CultureIndex.sol

185  for (uint i; i < creatorArrayLength; i++)

236    for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
        }
243 for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        } 
355   for (uint256 i; i < len; i++)     

403    for (uint256 i; i < len; i++) {
            if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();
        }

407        for (uint256 i; i < len; i++) {
            _voteForMany(pieceIds[i], from[i]);
        }
    }         
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L185
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L236
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L243
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L355
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L403-L407


```solidity

file: ERC20TokenEmitter.sol

209  for (uint256 i = 0; i < addresses.length; i++)


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L209

```solidity

file: AuctionHouse.sol

384   for (uint256 i = 0; i < numCreators; i++)
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L384

```solidity

file: VerbsToken.sol

306   for (uint i = 0; i < artPiece.creators.length; i++) {
                newPiece.creators.push(artPiece.creators[i]);
            }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L306

### [G-11] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

```solidity

file: AuctionHouse.sol

361   else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L361

### [G-12]  Not using the named return variable when a function returns, wastes deployment gas


```solidity

file: CultureIndex.sol

490   return pieceId;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L490

```solidity

file: VerbsToken.sol

314  return verbId;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L314

```solidity

file: abstract/RewardSplits.sol

91   return totalReward;


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L91