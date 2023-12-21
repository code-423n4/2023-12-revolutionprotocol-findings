## [L-01] check manager not zero in AuctionHouse constructor
### Old implementation
```solidity
    constructor(address _manager) payable initializer {
        manager = IRevolutionBuilder(_manager);
    }
```
### New implementation
```solidity
    constructor(address _manager) payable initializer {
        require(manager != address(0), "manager address is invalid");
        manager = IRevolutionBuilder(_manager);
    }
```


## [L-02] check MaxHeap address, is not zero in the CultureIndex initialize()
### Old implementation
```solidity
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
        require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");
        require(_erc721VotingToken != address(0), "invalid erc721 voting token");
        require(_erc20VotingToken != address(0), "invalid erc20 voting token");

        // Setup ownable
        __Ownable_init(_initialOwner);
--- code
        maxHeap = MaxHeap(_maxHeap);
    }

```
### New implementation
```solidity
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
        require(_cultureIndexParams.erc721VotingTokenWeight > 0, "invalid erc721 voting token weight");
        require(_erc721VotingToken != address(0), "invalid erc721 voting token");
        require(_erc20VotingToken != address(0), "invalid erc20 voting token");
        require(_maxHeap != address(0), "invalid MaxHeap address");
        // Setup ownable
        __Ownable_init(_initialOwner);
--- code
        maxHeap = MaxHeap(_maxHeap);
    }

```









## [N-01] Wrong Documentation on getPastVotes
### Old implementation
````solidity
    /**
     * @notice Returns the voting power of a voter at the current block.
     * @param account The address of the voter.
     * @return The voting power of the voter.
     */
    function getPastVotes(address account, uint256 blockNumber) external view override returns (uint256) {
        return _getPastVotes(account, blockNumber);
    }
````
### New implementation

````solidity
    /**
     * @notice Returns the voting power of a voter at a past block.
     * @param account The address of the voter.
     * @param uint256 The past block.
     * @return The voting power of the voter.
     */
    function getPastVotes(address account, uint256 blockNumber) external view override returns (uint256) {
        return _getPastVotes(account, blockNumber);
    }
````  

## [N-02] Wrong contract documentation on the NontransferableERC20Votes contract
In the contract documentation, it is specified that the ERC20 Token is transferable which is not true, this could lead to confusion.
```solidity
 * By default, token balance does not account for voting power. *This makes transfers cheaper*. The downside is that it
 * requires users to delegate to themselves in order to activate checkpoints and have their voting power tracked.
 */
```





