### [L-01] All ERC721 tokens have the same vote weight despite having different price

When an auction finishes successfully, an ERC721 is given to the auction winner. This ERC721 is the tokenized version of an art piece that had the most votes. However, an ERC721 piece is given the same vote weight as other ERC721 pieces. This means that if an ERC721 token is equal to 10 votes, then an ERC721 that costs (there is no inherent cost, just highest bid amount) 1 ETH, and another ERC721 that costs 0.0001 ETH both has 10 votes, which is unfair.

```
CultureIndex.sol
    function initialize(
        address _erc20VotingToken,
        address _erc721VotingToken,
        address _initialOwner,
        address _maxHeap,
        address _dropperAdmin,
        IRevolutionBuilder.CultureIndexParams memory _cultureIndexParams
    ) external initializer {
        require(msg.sender == address(manager), "Only manager can initialize");

        require(_cultureIndexParams.quorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "invalid quorum bps");
>       require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");
```

One possible solution to mitigate this is to give a multiplier to the ERC721. If the ERC721 token costs 10 ether, then give the token a 10x multiplier, so this token has 100 votes (1x multiplier for every 1 ether).

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L109-L122

### [L-02] The first piece created can pass quorumVotes without any votes if totalSupply of ERC20 votes is zero

If no ERC20tokens have been minted yet, the newPiece.quorumVotes will return zero.

```
        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
        newPiece.metadata = metadata;
        newPiece.sponsor = msg.sender;
        newPiece.creationBlock = block.number;
>       newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
```

```
   function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");

        ICultureIndex.ArtPiece memory piece = getTopVotedPiece();
>       require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");
```

It does not need any votes to pass and an ERC721 can immediately be minted and sent to auction.

Make sure that some vote tokens have been minted so that the quorumVotes will not be zero.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L523

### [L-03] Not all media types are checked for 0 values

Function `validateMediaType()` in CultureIndex.sol checks whether the length of certain MediaTypes have 0 value input

```
    function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

        if (metadata.mediaType == MediaType.IMAGE)
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        else if (metadata.mediaType == MediaType.ANIMATION)
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        else if (metadata.mediaType == MediaType.TEXT)
            require(bytes(metadata.text).length > 0, "Text must be provided");
    }
```

Some MediaTypes are not checked.

```
    enum MediaType {
        NONE, -> Not checked
        IMAGE, -> Checked
        ANIMATION, -> Checked
        AUDIO, -> Not checked
        TEXT, -> Checked
        OTHER -> Not checked
    }
```

Some Metadata are not checked either

```
    struct ArtPieceMetadata {
        string name; -> Not checked
        string description; -> Not checked
        MediaType mediaType;
        string image;
        string text;
        string animationUrl;
    }
```

Consider expanding the metadata so that all media types are checked for consistency. Otherwise, a person can just create a MediaType of audio, without a name or description or audio.

As stated in the requirements of creating an artPiece using `createPiece()`,

```
     * Requirements:
     * - `metadata` must include name, description, and image. Animation URL is optional.
    function createPiece(
```

A metadata can create a name and description as 0 value and still be created. Also, name and description can be repeated, not sure if intended.

Check all the inputs when validating metadata.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L159-L168
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L204-L209

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L284-L287

### [L-04] Temporary DoS of auction if computeTotalReward() is more than 50_000 ether or less than 0.0000001 ether

If entropyRateBps() is less than 10,000, erc20TokenEmitter.buyToken will be called with the remaining msg.value: { value: creatorsShare - ethPaidToCreators }. This value will be checked in `computeTotalReward()`, which is called by `_handleRewardsAndGetValueToSend`(). computeTotalReward() will check whether the msg.value is more than 50,000 ether or less than 0.0000001 ether.

```
    function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

```

It may be quite impossible to reach 50_000 ether, but 0.000001 ether is possible. In auction house, it checks whether there is any bidder for the auction. This implies that having no bidder is possible. An attacker can call a bid with 1 wei (provided reservePrice is 0) and pass this msg.value along to the erc20TokenEmitter.buyToken() function (owner and payment amount will truncate to zero).

```
      //If no one has bid, burn the Verb
            if (_auction.bidder == address(0))
                verbs.burn(_auction.verbId);
```

Make sure the min_bid is enforced to be a decent non-zero value (maybe >1e12) and make sure that msg.value is more than 0.0000001e18 before calling erc20TokenEmitter.buyToken.

```
AuctionHouse.sol
                 if (creatorsShare > ethPaidToCreators) {
+                    uint remainingAmt = creatorsShare - ethPaidToCreators;
+                    require(remainingAmt > 0.0000001e18);
+                    creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: remainingAmt }(

```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L399-L402

### [L-05] The treasury and creatorsAddress can skip the check in ERC20TokenEmitter.buyToken easily by transferring funds to another address and calling buyToken

In ERC20TokenEmitter.sol, there is a function called `buyToken()` where a buyer can buy vote tokens. There is a check that states the creatorAddress cannot buy the token. However, this check can be easily bypassed by the creatorAddress sending tokens to another address and calling buyToken.

```
    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        //prevent treasury from paying itself
>       require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L158C33-L158C33

### [L-06] Limit the amount of new pieces that can be created in CultureIndex to prevent high gas payments everytime a vote is casted

Whenever a new art piece is created in CultureIndex.sol, the piece is inserted into the maxHeap.

```
 function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

        // Validate the media type and associated data
        validateMediaType(metadata);

        uint256 pieceId = _currentPieceId++;

        /// @dev Insert the new piece into the max heap
>       maxHeap.insert(pieceId, 0);
```

There are no limitations as to how many pieces can be created. This means that anyone can create a new piece just by paying some gas fees. The heap can be constantly updated with new pieces, and it is assumed that there will always be more piece than auction available.

Note that the piece is inserted into the maxHeap with 0 as the default vote value. When a vote occurs, `updateValue()` is called, passing in the new weight value and checking whether the value is greater than the parent.

If the position is greater than the parent, do a swap. Also note the usage of a while loop.

```
        maxHeap.updateValue(pieceId, totalWeight);
```
```
function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
uint256 position = positionMapping[itemId];
uint256 oldValue = valueMapping[itemId];



        // Update the value in the valueMapping
        valueMapping[itemId] = newValue;

        // Decide whether to perform upwards or downwards heapify
        if (newValue > oldValue) {
            // Upwards heapify

>           while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
>               swap(position, parent(position));

                position = parent(position);
            }
        } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify
    }
```
Since there can be many thousand pieces that can exist at the same time without making the cut to go to the auction house, a user may have to spend a large amount of tokens just to cast a vote.

Even though using a heap uses O(1) time, there is still a gas limit. Limit the total number of art pieces that can coexist at the same time to prevent the heap from expanding too large.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L221
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L322

### [L-07] Voting weight for ERC721 tokens cannot be changed.

The voting weight of ERC721 tokens can only be set once, in the initializer. If there comes a point where the ERC721 tokens take up too much percentage of vote count, the owner cannot change the voting weight. For example, if one ERC721 token is set to 100 votes, it cannot be changed to 10 votes if too many ERC721 tokens are minted.

Consider having a setter function to change the ERC721 token voting weight

```
function initialize(
        address _erc20VotingToken,
        address _erc721VotingToken,
        address _initialOwner,
        address _maxHeap,
        address _dropperAdmin,
        IRevolutionBuilder.CultureIndexParams memory _cultureIndexParams
    ) external initializer {
       ...
      erc20VotingToken = ERC20VotesUpgradeable(_erc20VotingToken);
        erc721VotingToken = ERC721CheckpointableUpgradeable(_erc721VotingToken);
>        erc721VotingTokenWeight = _cultureIndexParams.erc721VotingTokenWeight;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L109-L134


### [L-08] MIN_TOKEN_MINT_GAS_THRESHOLD should be flexible

`MIN_TOKEN_MINT_GAS_THRESHOLD`is hardcoded as 750,000. This value should be changeable by the owner, although it may incur more centralization risk. Otherwise, remove this variable altogether.

```
    // TODO investigate this - The minimum gas threshold for creating an auction (minting VerbsToken)
    uint32 public constant MIN_TOKEN_MINT_GAS_THRESHOLD = 750_000;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L87-L88

### [I-01] If onboarding is slow, early users can monopolize the entire protocol

If the project have a stealth launch, the early users can 'control' the voting process by continuously voting their own piece as the top piece and claiming all the ERC721s (through auction, and the early users have a high probability to win the auction since there is not much participation to begin with). Since ERC721 has voting weight, their ERC721 that they win will net them more ERC721s with voting weight. If its not contained early, this situation will cascade in early users monopolizing on ERC721 tokens, and if the ERC721 voting weight is huge, then they will win all future voting as well.

Take note of this potential issue, and look out for early auctions with low participation. Worst case scenario, the protocol can participate in the auction to prevent such monopolization from occuring

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L171-L188

### [I-02] The usage of block.number on L2s may cause potential issues for future developments

block.number is not calculated the same on L2 as with L1.

On [Optimism](https://docs.optimism.io/chain/differences#block-numbers-and-timestamps), each transaction is it's own block. Same as [Base](https://www.coinbase.com/cloud/discover/protocol-guides/guide-to-base)

If the protocol considers tracking the block.number by implementing a blockDelay variable in the future to control the votes (eg within a certain block only x votes can be used), then do note that L1 blocks and L2 blocks have different values.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L233
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/VotesUpgradeable.sol#L97-L103

### [I-03] ERC20Votes cannot be transferred but ERC721 tokens with voting weight can be transferred

ERC20 votes have overriden functions that ban transfers and approvals. A person with ERC20 votes can only use the votes (not even burn it).

```
    function transfer(address, uint256) public virtual override returns (bool) {
        revert TRANSFER_NOT_ALLOWED();
    }
```

ERC721 votes have a voting weight as well, and the votes can be transferrable through delegation

```
 erc721VotingToken = ERC721CheckpointableUpgradeable(_erc721VotingToken);
```

```
    function _calculateVoteWeight(uint256 erc20Balance, uint256 erc721Balance) internal view returns (uint256) {
        return erc20Balance + (erc721Balance * erc721VotingTokenWeight * 1e18);
    }
```

Do note the discrepency of token transfers, unsure if this is intended.

A potential solution if this issue is unintended is to prevent ERC721 holders to transfer their ERC721 tokens or delegate their votes to another person

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L284-L287

### [I-04] ERC20Votes can technically be bought even if they are not transferrable or approved.

ERC20 vote token holders can sell their vote tokens indirectly by putting their votes up for sale, much like a gauge system. Vote holders can sell voting services to clients to circumvent the "no transfer/approval" restriction.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/NontransferableERC20Votes.sol#L94-L102
