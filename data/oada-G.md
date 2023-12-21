## [G-01]Remove initializer modifier
Remove the initializer modifier and call the `initialize()` function within the constructor to reduce gas consumption.
```solidity
packages\revolution\src\CultureIndex.sol

  116:    ) external initializer { 
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L116
```solidity
packages\revolution\src\VerbsToken.sol

  136:    ) external initializer { 
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L136
```solidity
packages\revolution\src\AuctionHouse.sol

  119:    ) external initializer { 
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L119C22-L119C22
## [G-02]No check for zero address
The constructor fails to check the zero address. If a zero address is passed in by mistake, the contract may be deployed again, consuming a large amount of gas.
```solidity
packages\revolution\src\CultureIndex.sol

  92:  constructor(address _manager) payable initializer {
  93:      manager = IRevolutionBuilder(_manager);
  94:  }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L92-L94
## [G-03]require() or revert() statements that check input arguments should be at the top of the function
Here are some issues not mentioned in the bot race.
```solidity
packages\revolution\src\CultureIndex.sol

  438:  if (from == address(0)) revert ADDRESS_ZERO();

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L438
## [G-04]Same value check should be added
In the following issues, the same value check should be added to prevent that when the same value is passed in by mistake, it will still run and waste gas.
```solidity
packages\revolution\src\CultureIndex.sol

    function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external onlyOwner {
+        require(newQuorumVotesBPS != quorumVotesBPS)
        require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");
        emit QuorumVotesBPSSet(quorumVotesBPS, newQuorumVotesBPS);

        quorumVotesBPS = newQuorumVotesBPS;
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L498-L503
```solidity
packages\revolution\src\VerbsToken.sol

    function setContractURIHash(string memory newContractURIHash) external onlyOwner {
+       require(newContractURIHash != _contractURIHash);
        _contractURIHash = newContractURIHash;
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L169-L171
```solidity
packages\revolution\src\VerbsToken.sol


    function setMinter(address _minter) external override onlyOwner nonReentrant whenMinterNotLocked {
+       require(_minter != minter);
        require(_minter != address(0), "Minter cannot be zero address");
        minter = _minter;

        emit MinterUpdated(_minter);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L209-L214
```solidity
packages\revolution\src\VerbsToken.sol


    function setDescriptor(
        IDescriptorMinimal _descriptor
    ) external override onlyOwner nonReentrant whenDescriptorNotLocked {
+       require(_descriptor != descriptor);
        descriptor = _descriptor;

        emit DescriptorUpdated(_descriptor);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L230-L236
```solidity
packages\revolution\src\VerbsToken.sol


    function setCultureIndex(ICultureIndex _cultureIndex) external onlyOwner whenCultureIndexNotLocked nonReentrant {
+       require(_cultureIndex != cultureIndex);
        cultureIndex = _cultureIndex;

        emit CultureIndexUpdated(_cultureIndex);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L252-L256
```solidity
packages\revolution\src\AuctionHouse.sol

    function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
+       require(_creatorRateBps != creatorRateBps);
        require(
            _creatorRateBps >= minCreatorRateBps,
            "Creator rate must be greater than or equal to minCreatorRateBps"
        );
        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
        creatorRateBps = _creatorRateBps;

        emit CreatorRateBpsUpdated(_creatorRateBps);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L217-L226
```solidity
packages\revolution\src\AuctionHouse.sol

    function setEntropyRateBps(uint256 _entropyRateBps) external onlyOwner {
+       require(_entropyRateBps != entropyRateBps);
        require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");

        entropyRateBps = _entropyRateBps;
        emit EntropyRateBpsUpdated(_entropyRateBps);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L253-L258
```solidity
packages\revolution\src\AuctionHouse.sol

    function setTimeBuffer(uint256 _timeBuffer) external override onlyOwner {
+       require(_timeBuffer != timeBuffer);
        timeBuffer = _timeBuffer;

        emit AuctionTimeBufferUpdated(_timeBuffer);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L277-L281
```solidity
packages\revolution\src\AuctionHouse.sol

    function setReservePrice(uint256 _reservePrice) external override onlyOwner {
+       require(_reservePrice != reservePrice);
        reservePrice = _reservePrice;

        emit AuctionReservePriceUpdated(_reservePrice);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L287-L291
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L297-L301