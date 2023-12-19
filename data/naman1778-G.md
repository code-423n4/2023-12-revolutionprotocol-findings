## [G-01] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

There is 1 instance of this issue in 1 file:

```
File: CultureIndex.sol	

431: voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));
```

    diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
    index 88f3338..a63d591 100644
    --- a/packages/revolution/src/CultureIndex.sol
    +++ b/packages/revolution/src/CultureIndex.sol
    @@ -428,7 +428,7 @@ contract CultureIndex is

             bytes32 voteHash;

    -        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));
    +        voteHash = keccak256(abi.encodePacked(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

             bytes32 digest = _hashTypedDataV4(voteHash);

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There is 1 instance of this issue in 1 file:

```
File: RewardSplits.sol	

30: if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");
```

    diff --git a/packages/protocol-rewards/src/abstract/RewardSplits.sol b/packages/protocol-rewards/src/abstract/RewardSplits.sol
    index c35ee26..1edf3e0 100644
    --- a/packages/protocol-rewards/src/abstract/RewardSplits.sol
    +++ b/packages/protocol-rewards/src/abstract/RewardSplits.sol
    @@ -27,7 +27,7 @@ abstract contract RewardSplits {
         IRevolutionProtocolRewards internal immutable protocolRewards;

         constructor(address _protocolRewards, address _revolutionRewardRecipient) payable {
    -        if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");
    +        if (((_protocolRewards ^ address(0)) & (_revolutionRewardRecipient ^ address(0))) == 0) revert("Invalid Address Zero");

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-03] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 9 instances of this issue in 4 files: 

```
File: CultureIndex.sol	

185: for (uint i; i < creatorArrayLength; i++) {

236: for (uint i; i < creatorArrayLength; i++) {

243: for (uint i; i < creatorArrayLength; i++) {

355: for (uint256 i; i < len; i++) {

403: for (uint256 i; i < len; i++) {

407: for (uint256 i; i < len; i++) {
```
    diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
    index 88f3338..99c522f 100644
    --- a/packages/revolution/src/CultureIndex.sol
    +++ b/packages/revolution/src/CultureIndex.sol
    @@ -182,7 +182,7 @@ contract CultureIndex is
             require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

             uint256 totalBps;
    -        for (uint i; i < creatorArrayLength; i++) {
    +        for (uint i = creatorArrayLength - 1; i >= 0; i--) {
                 require(creatorArray[i].creator != address(0), "Invalid creator address");
                 totalBps += creatorArray[i].bps;
             }
    @@ -233,14 +233,14 @@ contract CultureIndex is
             newPiece.creationBlock = block.number;
             newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;

    -        for (uint i; i < creatorArrayLength; i++) {
    +        for (uint i = creatorArrayLength - 1; i >= 0; i--) {
                 newPiece.creators.push(creatorArray[i]);
             }

             emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

             // Emit an event for each creator
    -        for (uint i; i < creatorArrayLength; i++) {
    +        for (uint i = creatorArrayLength  - 1; i >= 0; i--) {
                 emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
             }

    @@ -352,7 +352,7 @@ contract CultureIndex is
          */
         function _voteForMany(uint256[] calldata pieceIds, address from) internal {
             uint256 len = pieceIds.length;
    -        for (uint256 i; i < len; i++) {
    +        for (uint256 i = len - 1; i >= 0 ; i--) {
                 _vote(pieceIds[i], from);
             }
         }
    @@ -400,11 +400,11 @@ contract CultureIndex is
                 "Array lengths must match"
             );

    -        for (uint256 i; i < len; i++) {
    +        for (uint256 i = len - 1; i >= 0; i--) {
                 if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();
             }

    -        for (uint256 i; i < len; i++) {
    +        for (uint256 i = len - 1; i >= 0; i--) {
                 _voteForMany(pieceIds[i], from[i]);
             }
         }

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol

```
File: ERC20TokenEmitter.sol	

209: for (uint256 i = 0; i < addresses.length; i++) {
```

    diff --git a/packages/revolution/src/ERC20TokenEmitter.sol b/packages/revolution/src/ERC20TokenEmitter.sol
    index e6f7d46..0d4a36e 100644
    --- a/packages/revolution/src/ERC20TokenEmitter.sol
    +++ b/packages/revolution/src/ERC20TokenEmitter.sol
    @@ -206,7 +206,7 @@ contract ERC20TokenEmitter is

             //Mint tokens to buyers

    -        for (uint256 i = 0; i < addresses.length; i++) {
    +        for (uint256 i = addresses.length - 1; i >= 0 ; i--) {
                 if (totalTokensForBuyers > 0) {
                     // transfer tokens to address
                     _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol

```
File: AuctionHouse.sol	

384: for (uint256 i = 0; i < numCreators; i++) {
```
    diff --git a/packages/revolution/src/AuctionHouse.sol b/packages/revolution/src/AuctionHouse.sol
    index 55e55a1..4130959 100644
    --- a/packages/revolution/src/AuctionHouse.sol
    +++ b/packages/revolution/src/AuctionHouse.sol
    @@ -381,7 +381,7 @@ contract AuctionHouse is

                     //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                     if (creatorsShare > 0 && entropyRateBps > 0) {
    -                    for (uint256 i = 0; i < numCreators; i++) {
    +                    for (uint256 i = numCreators - 1; i >= 0 ; i--) {
                             ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
                             vrgdaReceivers[i] = creator.creator;
                             vrgdaSplits[i] = creator.bps;

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol

```
File: VerbsToken.sol	
    
306: for (uint i = 0; i < artPiece.creators.length; i++) {
```

    diff --git a/packages/revolution/src/VerbsToken.sol b/packages/revolution/src/VerbsToken.sol
    index 7bd9527..5dbb107 100644
    --- a/packages/revolution/src/VerbsToken.sol
    +++ b/packages/revolution/src/VerbsToken.sol
    @@ -303,7 +303,7 @@ contract VerbsToken is
                 newPiece.quorumVotes = artPiece.quorumVotes;
                 newPiece.totalVotesSupply = artPiece.totalVotesSupply;

    -            for (uint i = 0; i < artPiece.creators.length; i++) {
    +            for (uint i = artPiece.creators.length - 1; i >= 0 ; i--) {
                     newPiece.creators.push(artPiece.creators[i]);
                 }

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol

#### Test Code

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

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-04] Use assembly to write address storage values

When writing value for variables whose type is address, make use of assembly code instead of solidity code.

There are 7 instances of this issue in 6 files: 

```
File: MaxHeap.sol	

58: admin = _admin;
```

    diff --git a/packages/revolution/src/MaxHeap.sol b/packages/revolution/src/MaxHeap.sol
    index 5dfb81c..2a202ad 100644
    --- a/packages/revolution/src/MaxHeap.sol
    +++ b/packages/revolution/src/MaxHeap.sol
    @@ -55,7 +55,9 @@ contract MaxHeap is VersionedContract, UUPS, Ownable2StepUpgradeable, Reentrancy
         function initialize(address _initialOwner, address _admin) public initializer {
             require(msg.sender == address(manager), "Only manager can initialize");

    -        admin = _admin;
    +       assembly{
    +               sstore(admin,_admin)
    +        }

             __Ownable_init(_initialOwner);
             __ReentrancyGuard_init();

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol

```
File: CultureIndex.sol	

139: dropperAdmin = _dropperAdmin;
```

    diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
    index 88f3338..64e2e61 100644
    --- a/packages/revolution/src/CultureIndex.sol
    +++ b/packages/revolution/src/CultureIndex.sol
    @@ -136,8 +136,9 @@ contract CultureIndex is
             description = _cultureIndexParams.description;
             quorumVotesBPS = _cultureIndexParams.quorumVotesBPS;
             minVoteWeight = _cultureIndexParams.minVoteWeight;
    -        dropperAdmin = _dropperAdmin;
    -
    +        assembly{
    +            sstore(dropperAdmin,_dropperAdmin)
    +        }
             emit QuorumVotesBPSSet(quorumVotesBPS, _cultureIndexParams.quorumVotesBPS);

             // Create maxHeap

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol

```
File: ERC20TokenEmitter.sol	

101: treasury = _treasury;
```

    diff --git a/packages/revolution/src/ERC20TokenEmitter.sol b/packages/revolution/src/ERC20TokenEmitter.sol
    index e6f7d46..425f630 100644
    --- a/packages/revolution/src/ERC20TokenEmitter.sol
    +++ b/packages/revolution/src/ERC20TokenEmitter.sol
    @@ -98,7 +98,9 @@ contract ERC20TokenEmitter is
             // Set up ownable
             __Ownable_init(_initialOwner);
    
    -        treasury = _treasury;
    +        assembly{
    +            sstore(treasury,_treasury)
    +        }
             creatorsAddress = _creatorsAddress;
             vrgdac = VRGDAC(_vrgdac);
             token = NontransferableERC20Votes(_erc20Token);

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol

```
File: AuctionHouse.sol	

143: WETH = _weth;
```
    diff --git a/packages/revolution/src/AuctionHouse.sol b/packages/revolution/src/AuctionHouse.sol
    index 55e55a1..3ca50c5 100644
    --- a/packages/revolution/src/AuctionHouse.sol
    +++ b/packages/revolution/src/AuctionHouse.sol
    @@ -140,7 +140,9 @@ contract AuctionHouse is
             creatorRateBps = _auctionParams.creatorRateBps;
             entropyRateBps = _auctionParams.entropyRateBps;
             minCreatorRateBps = _auctionParams.minCreatorRateBps;
    -        WETH = _weth;
    +        assembly{
    +            sstore(WETH,_weth)
    +        }
         }

         /**

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol

```
File: VerbsToken.sol	

153: minter = _minter;

211: minter = _minter;
```
    diff --git a/packages/revolution/src/VerbsToken.sol b/packages/revolution/src/VerbsToken.sol
    index 7bd9527..87dc464 100644
    --- a/packages/revolution/src/VerbsToken.sol
    +++ b/packages/revolution/src/VerbsToken.sol
    @@ -150,7 +150,9 @@ contract VerbsToken is
             _contractURIHash = _erc721TokenParams.contractURIHash;

             // Set the contracts
    -        minter = _minter;
    +        assembly{
    +            sstore(minter,_minter)
    +        }
             descriptor = IDescriptorMinimal(_descriptor);
             cultureIndex = ICultureIndex(_cultureIndex);
         }
    @@ -208,8 +210,9 @@ contract VerbsToken is
          */
         function setMinter(address _minter) external override onlyOwner nonReentrant whenMinterNotLocked {
             require(_minter != address(0), "Minter cannot be zero address");
    -        minter = _minter;
    -
    +        assembly{
    +            sstore(minter,_minter)
    +        }
             emit MinterUpdated(_minter);
         }

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol

```
File: RewardSplits.sol	

33: revolutionRewardRecipient = _revolutionRewardRecipient;
```

    diff --git a/packages/protocol-rewards/src/abstract/RewardSplits.sol b/packages/protocol-rewards/src/abstract/RewardSplits.sol
    index c35ee26..42c663c 100644
    --- a/packages/protocol-rewards/src/abstract/RewardSplits.sol
    +++ b/packages/protocol-rewards/src/abstract/RewardSplits.sol
    @@ -30,7 +30,9 @@ abstract contract RewardSplits {
             if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");

             protocolRewards = IRevolutionProtocolRewards(_protocolRewards);
    -        revolutionRewardRecipient = _revolutionRewardRecipient;
    +        assembly{
    +            sstore(revolutionRewardRecipient,_revolutionRewardRecipient)
    +        }
         }

         /*

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }
    
    contract Contract0 {
    
        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }
    
    }
    
    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }
    
    }

#### Gas Report

|  Contract0 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |

|  Contract1 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |