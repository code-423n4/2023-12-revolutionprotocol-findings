#  Gas Optimizations

| Number | 	Issue | nstances |
|--------|--------|----------|
|[G-01]| Avoid Unnecessary Public Variables  | 44 |
|[G-02]| State variables that are used multiple times in a function should be cached in stack variables  | 13 |
|[G-03]| Use assembly to write address storage values  | 7 |
|[G-04]| Increments can be unchecked in for-loops  | 9 |
|[G-05]| Use named returns for local variables of pure functions where it is possible  | 3 |
|[G-06]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 10 |
|[G-07]| require() statements that check input arguments should be at the top of the function  | 2 |
|[G-08]| Redundant state variable getters  | 3 |
|[G-09]| CultureIndex._verifyVoteSignature() function can be optimized  | 1 |
|[G-10]| Don’t cache calls/variables if using them once  | 2 |
|[G-11]| Using storage instead of memory for structs/arrays saves gas  | 1 |
|[G-12]| (Proposal) Variables that should be constant/immutable  | 10 |
|[G-13]| For same condition checks use modifiers  | 4 |
|[G-14]| Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient  | 5 |
|[G-15]| Counting down in for statements is more gas efficient  | 8 |
|[G-16]| Use assembly to perform efficient back-to-back calls  | 1 |
|[G-17]| Declare the variables outside the loop  | 1 |
|[G-18]| Explicitly assigning a default value in storage wastes gas  | 4 |


## [G-01] Avoid Unnecessary Public Variables

Public storage variables increase the contract's size due to the implicit generation of public getter functions. This makes the contract larger and could increase deployment and interaction costs.

If you do not require other contracts to read these variables, consider making them private or internal.

```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

48        IVerbsToken public verbs;

    // The ERC20 governance token
    IERC20TokenEmitter public erc20TokenEmitter;

    // The address of the WETH contract
    address public WETH;

    // The minimum amount of time left in an auction after a new bid is created
    uint256 public timeBuffer;

    // The minimum price accepted in an auction
    uint256 public reservePrice;

    // The minimum percentage difference between the last bid amount and the current bid
    uint8 public minBidIncrementPercentage;

    // The split of the winning bid that is reserved for the creator of the Verb in basis points
    uint256 public creatorRateBps;

    // The all time minimum split of the winning bid that is reserved for the creator of the Verb in basis points
    uint256 public minCreatorRateBps;

    // The split of (auction proceeds * creatorRate) that is sent to the creator as ether in basis points
    uint256 public entropyRateBps;

    // The duration of a single auction
    uint256 public duration;

    // The active auction
    IAuctionHouse.Auction public auction;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L48-L78


```solidity
file:  main/packages/revolution/src/CultureIndex.sol

36       MaxHeap public maxHeap;

    // The ERC20 token used for voting
    ERC20VotesUpgradeable public erc20VotingToken;

    // The ERC721 token used for voting
    ERC721CheckpointableUpgradeable public erc721VotingToken;

    // The weight of the 721 voting token
    uint256 public erc721VotingTokenWeight;

    /// @notice The maximum settable quorum votes basis points
    uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%

    /// @notice The minimum vote weight required in order to vote
    uint256 public minVoteWeight;

    /// @notice The basis point number of votes in support of a art piece required in order for a quorum to be reached and for an art piece to be dropped.
    uint256 public quorumVotesBPS;

    /// @notice The name of the culture index
    string public name;

    /// @notice A description of the culture index - can include rules or guidelines
    string public description;

    // The list of all pieces
    mapping(uint256 => ArtPiece) public pieces;

    // The internal piece ID tracker
    uint256 public _currentPieceId;

    // The mapping of all votes for a piece
    mapping(uint256 => mapping(address => Vote)) public votes;

    // The total voting weight for a piece
    mapping(uint256 => uint256) public totalVoteWeights;

    // Constant for max number of creators
    uint256 public constant MAX_NUM_CREATORS = 100;

    // The address that is allowed to drop art pieces
    address public dropperAdmin;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L36-L78


```solidity
file:   main/packages/revolution/src/ERC20TokenEmitter.sol

25       address public treasury;

    // The token that is being minted.
    NontransferableERC20Votes public token;

    // The VRGDA contract
    VRGDAC public vrgdac;

    // solhint-disable-next-line not-rely-on-time
    uint256 public startTime;

    /**
     * @notice A running total of the amount of tokens emitted.
     */
    int256 public emittedTokenWad;

    // The split of the purchase that is reserved for the creator of the Verb in basis points
    uint256 public creatorRateBps;

    // The split of (purchase proceeds * creatorRate) that is sent to the creator as ether in basis points
    uint256 public entropyRateBps;

    // The account or contract to pay the creator reward to
    address public creatorsAddress;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L25-L48


```solidity
file:   main/packages/revolution/src/VerbsToken.sol

41       // An address who has permissions to mint Verbs
    address public minter;

    // The Verbs token URI descriptor
    IDescriptorMinimal public descriptor;

    // The CultureIndex contract
    ICultureIndex public cultureIndex;

    // Whether the minter can be updated
    bool public isMinterLocked;

    // Whether the CultureIndex can be updated
    bool public isCultureIndexLocked;

    // Whether the descriptor can be updated
    bool public isDescriptorLocked;

    // The internal verb ID tracker
    uint256 private _currentVerbId;

    // IPFS content hash of contract-level metadata
    string private _contractURIHash = "QmQzDwaZ7yQxHHs7sQQenJVB89riTSacSGcJRv9jtHPuz5";

    // The Verb art pieces
    mapping(uint256 => ICultureIndex.ArtPiece) public artPieces;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L41-L66

## [G-02] State variables that are used multiple times in a function should be cached in stack variables

By caching state variables in stack variables, we reduce the need to frequently access storage, thereby saving gas.

### in function _settleAuction() cache the verbs variable instead of access multiple time from storage 

```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

336       function _settleAuction() internal {
        IAuctionHouse.Auction memory _auction = auction;

        require(_auction.startTime != 0, "Auction hasn't begun");
        require(!_auction.settled, "Auction has already been settled");
        //slither-disable-next-line timestamp
        require(block.timestamp >= _auction.endTime, "Auction hasn't completed");

        auction.settled = true;

        uint256 creatorTokensEmitted = 0;
        // Check if contract balance is greater than reserve price
        if (address(this).balance < reservePrice) {
            // If contract balance is less than reserve price, refund to the last bidder
            if (_auction.bidder != address(0)) {
                _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
            }

            // And then burn the Noun
            verbs.burn(_auction.verbId);
        } else {
            //If no one has bid, burn the Verb
            if (_auction.bidder == address(0))
                verbs.burn(_auction.verbId);
                //If someone has bid, transfer the Verb to the winning bidder
            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

            if (_auction.amount > 0) {
                // Ether going to owner of the auction
                uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;

                //Total amount of ether going to creator
                uint256 creatorsShare = _auction.amount - auctioneerPayment;

                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

                //Build arrays for erc20TokenEmitter.buyToken
                uint256[] memory vrgdaSplits = new uint256[](numCreators);
                address[] memory vrgdaReceivers = new address[](numCreators);

                //Transfer auction amount to the DAO treasury
                _safeTransferETHWithFallback(owner(), auctioneerPayment);

                uint256 ethPaidToCreators = 0;

                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                if (creatorsShare > 0 && entropyRateBps > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L336-L385

### in function createBid() and _settleAuction() cache the auction variable instead of access multiple time from storage 

```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

171    function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
        IAuctionHouse.Auction memory _auction = auction;

        //require bidder is valid address
        require(bidder != address(0), "Bidder cannot be zero address");
        require(_auction.verbId == verbId, "Verb not up for auction");
        //slither-disable-next-line timestamp
        require(block.timestamp < _auction.endTime, "Auction expired");
        require(msg.value >= reservePrice, "Must send at least reservePrice");
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );

        address payable lastBidder = _auction.bidder;

        auction.amount = msg.value;
        auction.bidder = payable(bidder);

        // Extend the auction if the bid was received within `timeBuffer` of the auction end time
        bool extended = _auction.endTime - block.timestamp < timeBuffer;
        if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;

        // Refund the last bidder, if applicable
        if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);

        emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);

        if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);
    }

336      function _settleAuction() internal {
        IAuctionHouse.Auction memory _auction = auction;

        require(_auction.startTime != 0, "Auction hasn't begun");
        require(!_auction.settled, "Auction has already been settled");
        //slither-disable-next-line timestamp
        require(block.timestamp >= _auction.endTime, "Auction hasn't completed");

        auction.settled = true;

        uint256 creatorTokensEmitted = 0;
        // Check if contract balance is greater than reserve price
        if (address(this).balance < reservePrice) {
            // If contract balance is less than reserve price, refund to the last bidder
            if (_auction.bidder != address(0)) {
                _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
            }

            // And then burn the Noun
            verbs.burn(_auction.verbId);
        } else {
            //If no one has bid, burn the Verb
            if (_auction.bidder == address(0))
                verbs.burn(_auction.verbId);
                //If someone has bid, transfer the Verb to the winning bidder
            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

            if (_auction.amount > 0) {
                // Ether going to owner of the auction
                uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;

                //Total amount of ether going to creator
                uint256 creatorsShare = _auction.amount - auctioneerPayment;

                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

                //Build arrays for erc20TokenEmitter.buyToken
                uint256[] memory vrgdaSplits = new uint256[](numCreators);
                address[] memory vrgdaReceivers = new address[](numCreators);

                //Transfer auction amount to the DAO treasury
                _safeTransferETHWithFallback(owner(), auctioneerPayment);

                uint256 ethPaidToCreators = 0;

                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                if (creatorsShare > 0 && entropyRateBps > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
                        vrgdaReceivers[i] = creator.creator;
                        vrgdaSplits[i] = creator.bps;

                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
                    }
                }

                //Buy token from ERC20TokenEmitter for all the creators
                if (creatorsShare > ethPaidToCreators) {
                    creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: creatorsShare - ethPaidToCreators }(
                        vrgdaReceivers,
                        vrgdaSplits,
                        IERC20TokenEmitter.ProtocolRewardAddresses({
                            builder: address(0),
                            purchaseReferral: address(0),
                            deployer: deployer
                        })
                    );
                }
            }
        }

        emit AuctionSettled(_auction.verbId, _auction.bidder, _auction.amount, creatorTokensEmitted);
    }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L171-L200

### in function initialize() and createBid cache the timeBuffer variable instead of access multiple time from storage 


```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

113     function initialize(
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


171  function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
        IAuctionHouse.Auction memory _auction = auction;

        //require bidder is valid address
        require(bidder != address(0), "Bidder cannot be zero address");
        require(_auction.verbId == verbId, "Verb not up for auction");
        //slither-disable-next-line timestamp
        require(block.timestamp < _auction.endTime, "Auction expired");
        require(msg.value >= reservePrice, "Must send at least reservePrice");
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );

        address payable lastBidder = _auction.bidder;

        auction.amount = msg.value;
        auction.bidder = payable(bidder);

        // Extend the auction if the bid was received within `timeBuffer` of the auction end time
        bool extended = _auction.endTime - block.timestamp < timeBuffer;
        if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;

        // Refund the last bidder, if applicable
        if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);

        emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);

        if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);
    }


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L113-L144

### in function initialize() cache the reservePrice variable instead of access multiple time from storage 

```solidity
file:  main/packages/revolution/src/AuctionHouse.sol
 
137   reservePrice = _auctionParams.reservePrice;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L137

### in function initialize() cache the reservePrice variable instead of access multiple time from storage 

```solidity
file: main/packages/revolution/src/AuctionHouse.sol
 
138   minBidIncrementPercentage = _auctionParams.minBidIncrementPercentage;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L138

### in function initialize() cache the minCreatorRateBps variable instead of access multiple time from storage 

```solidity
file: main/packages/revolution/src/AuctionHouse.sol

142   minCreatorRateBps = _auctionParams.minCreatorRateBps;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L142

### in function initialize() cache the erc721VotingTokenWeight variable instead of access multiple time from storage  

```solidity
file:  main/packages/revolution/src/CultureIndex.sol

119           require(_cultureIndexParams.quorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "invalid quorum bps");
        require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");
        require(_erc721VotingToken != address(0), "invalid erc721 voting token");
        require(_erc20VotingToken != address(0), "invalid erc20 voting token");

        // Setup ownable
        __Ownable_init(_initialOwner);

        // Initialize EIP-712 support
        __EIP712_init(string.concat(_cultureIndexParams.name, " CultureIndex"), "1");

        __ReentrancyGuard_init();

        erc20VotingToken = ERC20VotesUpgradeable(_erc20VotingToken);
        erc721VotingToken = ERC721CheckpointableUpgradeable(_erc721VotingToken);
        erc721VotingTokenWeight = _cultureIndexParams.erc721VotingTokenWeight;


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L119-L134

### in function _setQuorumVotesBPS() cache the quorumVotesBPS variable instead of access multiple time from storage   

```solidity
file: main/packages/revolution/src/CultureIndex.sol

490   function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external onlyOwner {
        require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");
        emit QuorumVotesBPSSet(quorumVotesBPS, newQuorumVotesBPS);

        quorumVotesBPS = newQuorumVotesBPS;
    }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L490-503


### in function initialize() cache the name variable instead of access multiple time from storage  

```solidity
file:   main/packages/revolution/src/CultureIndex.sol

109       function initialize(
        address _erc20VotingToken,
        address _erc721VotingToken,
        address _initialOwner,
        address _maxHeap,
        address _dropperAdmin,
        IRevolutionBuilder.CultureIndexParams memory _cultureIndexParams
    ) external initializer {
        require(msg.sender == address(manager), "Only manager can initialize");

        require(_cultureIndexParams.quorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "invalid quorum bps");
        require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");
        require(_erc721VotingToken != address(0), "invalid erc721 voting token");
        require(_erc20VotingToken != address(0), "invalid erc20 voting token");

        // Setup ownable
        __Ownable_init(_initialOwner);

        // Initialize EIP-712 support
        __EIP712_init(string.concat(_cultureIndexParams.name, " CultureIndex"), "1");

        __ReentrancyGuard_init();

        erc20VotingToken = ERC20VotesUpgradeable(_erc20VotingToken);
        erc721VotingToken = ERC721CheckpointableUpgradeable(_erc721VotingToken);
        erc721VotingTokenWeight = _cultureIndexParams.erc721VotingTokenWeight;
        name = _cultureIndexParams.name;
        description = _cultureIndexParams.description;
        quorumVotesBPS = _cultureIndexParams.quorumVotesBPS;
        minVoteWeight = _cultureIndexParams.minVoteWeight;
        dropperAdmin = _dropperAdmin;

        emit QuorumVotesBPSSet(quorumVotesBPS, _cultureIndexParams.quorumVotesBPS);

        // Create maxHeap
        maxHeap = MaxHeap(_maxHeap);
    }


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L109-L145


### in function _vote() cache the totalVoteWeights variable instead of access multiple time from storage  

```solidity
file: main/packages/revolution/src/CultureIndex.sol

307  function _vote(uint256 pieceId, address voter) internal {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        require(voter != address(0), "Invalid voter address");
        require(!pieces[pieceId].isDropped, "Piece has already been dropped");
        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");

        uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);
        require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");

        votes[pieceId][voter] = Vote(voter, weight);
        totalVoteWeights[pieceId] += weight;

        uint256 totalWeight = totalVoteWeights[pieceId];

        // TODO add security consideration here based on block created to prevent flash attacks on drops?
        maxHeap.updateValue(pieceId, totalWeight);
        emit VoteCast(pieceId, voter, weight, totalWeight);
    }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L307-324

### in function buyToken() cache the creatorsAddress variable instead of access multiple time from storage  

```solidity
file: main/packages/revolution/src/ERC20TokenEmitter.sol

195   if (creatorDirectPayment > 0) {
            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed.");
        }

        //Mint tokens for creators
        if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
            _mint(creatorsAddress, uint256(totalTokensForCreators));
        }


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L195-L203

## [G-03] Use assembly to write address storage values

```solidity
file:  main/packages/revolution/src/ERC20TokenEmitter.sol

101     treasury = _treasury;

102     creatorsAddress = _creatorsAddress;

103     vrgdac = VRGDAC(_vrgdac);

104     token = NontransferableERC20Votes(_erc20Token);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L101


```solidity
file:  main/packages/revolution/src/VerbsToken.sol

153     minter = _minter;

154     descriptor = IDescriptorMinimal(_descriptor);

155     cultureIndex = ICultureIndex(_cultureIndex);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L153

## [G-04] Increments can be unchecked in for-loops

This one isn’t covered by c4udit

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

ethereum/solidity#10695

Consider wrapping with an unchecked block here (around 25 gas saved per instance):

```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

384   for (uint256 i = 0; i < numCreators; i++) {
    
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L384


```solidity
file: main/packages/revolution/src/CultureIndex.sol

185   for (uint i; i < creatorArrayLength; i++) {

236   for (uint i; i < creatorArrayLength; i++) {

243   for (uint i; i < creatorArrayLength; i++) {

355   for (uint256 i; i < len; i++) {

403   for (uint256 i; i < len; i++) {

407   for (uint256 i; i < len; i++) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L185

```solidity
file:   main/packages/revolution/src/ERC20TokenEmitter.sol

209    for (uint256 i = 0; i < addresses.length; i++) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L209

```solidity
file: main/packages/revolution/src/VerbsToken.sol

306   for (uint i = 0; i < artPiece.creators.length; i++) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L306


## [G-05] Use named returns for local variables of pure functions where it is possible

```solidity
file: main/packages/revolution/src/MaxHeap.sol

78    function parent(uint256 pos) private pure returns (uint256) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L78


```solidity
file: main/packages/protocol-rewards/src/abstract/RewardSplits.sol

40   function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40

```solidity
file: main/packages/revolution/src/CultureIndex.sol

179   function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L179

## [G-06] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

```solidity
file: main/packages/revolution/src/VerbsToken.sol

51   bool public isMinterLocked;

54   bool public isCultureIndexLocked;

57   bool public isDescriptorLocked;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L51

```solidity
file: main/packages/revolution/src/AuctionHouse.sol

191   bool extended

424   bool success;

438   bool wethSuccess

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L191


```solidity
file: main/packages/revolution/src/AuctionHouse.sol

344   auction.settled = true;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L344

```solidity
file:  main/packages/revolution/src/VerbsToken.sol

221    isMinterLocked = true;

243    isDescriptorLocked = true;

263    isCultureIndexLocked = true;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L221

## [G-07] require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a SLOAD (2100 gas for the 1st one) in a function that may ultimately revert in the unhappy case.

```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

123    __Pausable_init();
        __ReentrancyGuard_init();
        __Ownable_init(_initialOwner);

        _pause();

        require(
            _auctionParams.creatorRateBps >= _auctionParams.minCreatorRateBps,
            "Creator rate must be greater than or equal to the creator rate"
        );

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L123-L132

### Recommanded code 

```solidity
file:
  
     require(
            _auctionParams.creatorRateBps >= _auctionParams.minCreatorRateBps,
            "Creator rate must be greater than or equal to the creator rate"
        );

    __Pausable_init();
        __ReentrancyGuard_init();
        __Ownable_init(_initialOwner);

        _pause();


```

```solidity
file:   main/packages/revolution/src/ERC20TokenEmitter.sol

93           __Pausable_init();
        __ReentrancyGuard_init();

        require(_treasury != address(0), "Invalid treasury address");

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L93-L96


### Recommanded code 


```solidity

require(_treasury != address(0), "Invalid treasury address");

__Pausable_init();
        __ReentrancyGuard_init();

```

## [G-08] Redundant state variable getters

Getters for public state variables are automatically generated by the solidity compiler so there is no need to code them manually as this increases deployment cost.

1. Make _getVotes mapping variable private or internal since a getter function was defined for it.

```solidity
file: main/packages/revolution/src/CultureIndex.sol

265  function getVotes(address account) external view override returns (uint256) {
        return _getVotes(account);
    }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L265-L267

The solidity compiler would automatically create a getter function for the _getVotes mapping above since it is declared as a public variable. However, a getter function getVotes() was also declared in the contract for the same variable; thereby, making two getter functions for the same variable in the contract. We could rectify this issue by making the _getVotes variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

3. Make getArtPieceById mapping variable private or internal since a hasVoted function was defined for it.

```solidity
file:  main/packages/revolution/src/VerbsToken.sol

273   function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
        require(verbId <= _currentVerbId, "Invalid piece ID");
        return artPieces[verbId];
    }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L273-L276

2. Make votes mapping variable private or internal since a hasVoted function was defined for it.

```solidity
file: main/packages/revolution/src/CultureIndex.sol

256   function hasVoted(uint256 pieceId, address voter) external view returns (bool) {
        return votes[pieceId][voter].voterAddress != address(0);
    }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L256-L258


## [G-09] CultureIndex._verifyVoteSignature() function can be optimized

By moving condition checks up in the execution flow, we save on computational steps and improve the gas efficiency of the _verifyVoteSignature () function.

```solidity
file:  main/packages/revolution/src/CultureIndex.sol

429     bytes32 voteHash;

        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

        bytes32 digest = _hashTypedDataV4(voteHash);

        address recoveredAddress = ecrecover(digest, v, r, s);

        // Ensure to address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();


```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L429-L438

## [G-10] Don’t cache calls/variables if using them once

No need to cache 
_hashTypedDataV4(voteHash)

```solidity
file: main/packages/revolution/src/CultureIndex.sol

433  bytes32 digest = _hashTypedDataV4(voteHash);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L433

No need to cache 
_verifyVoteSignature(from, pieceIds, deadline, v, r, s);

```solidity
file: main/packages/revolution/src/CultureIndex.sol
 
375   bool success = _verifyVoteSignature(from, pieceIds, deadline, v, r, s);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L375


## [G‑11] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

There is 1 instance of this issue:

```solidity
file: main/packages/protocol-rewards/src/abstract/RewardSplits.sol

72   (RewardsSettings memory settings, uint256 totalReward) = computePurchaseRewards(paymentAmountWei);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L72


## [G-12] (Proposal) Variables that should be constant/immutable

There are some variables that are very unlikely to change

```solidity
file: main/packages/revolution/src/ERC20TokenEmitter.sol

28    NontransferableERC20Votes public token;

31    VRGDAC public vrgdac;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L28

```solidity
file: main/packages/revolution/src/VerbsToken.sol

45   IDescriptorMinimal public descriptor;

48   ICultureIndex public cultureIndex;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L45

```solidity
file: main/packages/revolution/src/AuctionHouse.sol

48   IVerbsToken public verbs;

51   IERC20TokenEmitter public erc20TokenEmitter;

78   IAuctionHouse.Auction public auction;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L48


```solidity
file:  main/packages/revolution/src/CultureIndex.sol

36    MaxHeap public maxHeap;

39    ERC20VotesUpgradeable public erc20VotingToken;

42    ERC721CheckpointableUpgradeable public erc721VotingToken;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L36

## [G-13] For same condition checks use modifiers

```solidity
file: main/packages/revolution/src/CultureIndex.sol

308  require(pieceId < _currentPieceId, "Invalid piece ID");

452  require(pieceId < _currentPieceId, "Invalid piece ID");

462  require(pieceId < _currentPieceId, "Invalid piece ID");

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L308

```solidity
file:  packages/revolution/src/ERC20TokenEmitter.sol

192    require(success, "Transfer failed.");

197    require(success, "Transfer failed.");

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L192

```solidity
file: main/packages/revolution/src/MaxHeap.sol

157   require(size > 0, "Heap is empty");

170   require(size > 0, "Heap is empty");

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L157

```solidity
file:  main/packages/revolution/src/VerbsToken.sol

139    require(_minter != address(0), "Minter cannot be zero address");

210    require(_minter != address(0), "Minter cannot be zero address");

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L139

## [G-14] Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient

SSTORE from 0 to 1 (or any non-zero value) costs 20000 gas. SSTORE from 1 to 2 (or any other non-zero value) costs 5000 gas.

By storing the original value once again, a refund is triggered (https://eips.ethereum.org/EIPS/eip-2200).

Since refunds are capped to a percentage of the total transaction’s gas, it is best to keep them low, to increase the likelihood of the full refund coming into effect.

Therefore, switching between 1, 2 instead of 0, 1 will be more gas efficient.

```solidity
file: main/packages/revolution/src/AuctionHouse.sol

344  auction.settled = true;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L344


```solidity
file: main/packages/revolution/src/CultureIndex.sol

526   pieces[piece.pieceId].isDropped = true;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L526


```solidity
file:  main/packages/revolution/src/VerbsToken.sol
 
221   isMinterLocked = true;

243   isDescriptorLocked = true;

263   isCultureIndexLocked = true;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L221

## [G‑15] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

by changing this logic you can save 12171 gas per one for loop 

Tools used Remix


```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

384    for (uint256 i = 0; i < numCreators; i++) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L384

```solidity
file:  main/packages/revolution/src/CultureIndex.sol

185   for (uint i; i < creatorArrayLength; i++) {

236   for (uint i; i < creatorArrayLength; i++) {

243   for (uint i; i < creatorArrayLength; i++) {

355   for (uint256 i; i < len; i++) {

403   for (uint256 i; i < len; i++) {

407   for (uint256 i; i < len; i++) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L185


```solidity
file: main/packages/revolution/src/ERC20TokenEmitter.sol

209   for (uint256 i = 0; i < addresses.length; i++) {

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L209

### Test Code

```solidity
contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}
```

## [G-16] Use assembly to perform efficient back-to-back calls

```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

370    uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
371    address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L370-L371

## [G-17] Declare the variables outside the loop

Per iterations saves 26 GAS

```solidity
file:  main/packages/revolution/src/AuctionHouse.sol

390    uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L390


## [G-18] Explicitly assigning a default value in storage wastes gas

This is a useless storage writing as it assigns the default value:

```solidity
file: main/packages/revolution/src/MaxHeap.sol

67   uint256 public size = 0;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L67

```solidity
file: main/packages/revolution/src/AuctionHouse.sol

346   uint256 creatorTokensEmitted = 0;

380   uint256 ethPaidToCreators = 0;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L346


```solidity
file:  main/packages/revolution/src/ERC20TokenEmitter.sol

205   uint256 bpsSum = 0;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L205