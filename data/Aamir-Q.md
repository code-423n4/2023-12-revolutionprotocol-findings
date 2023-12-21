# Findings Summary

| |Issue|Instances|
|-|:-|:-:|
| [[L-0](#low-0)] | Incorrect or Incomplete Natspac Comments | 3 |
| [[L-1](#low-1)] | Creating a new Piece when ERC20 and ERC721 token supply is zero will cause issues | 1 |
| [[L-2](#low-2)] | Setting `Auction::reservePrice` equal to 0 can create a chain of 0 price bids | 1 |
| [[L-3](#low-3)] | `Auction::_settleAuction()` can cause DoS if the remaining amount sent to `ERCTokenEmitter::buyToken()` is less than `RewardsSplit::minPurchaseAmount` | 1 |
| [[L-4](#low-4)] | `ERC20TokenEmitter` allow for buying token by passing deployer, builder and purchase refferal to his own address | 1 |
| [[L-5](#low-5)] | `MaxHeap` is inheriting from `ReentrancyGuardUpgradeable`, but non reentrant check is not done anywhere in the contract | 1 |
| [[L-6](#low-6)] | `MaxHeap::insert` does not check if the value already exists for particular id and overwrites it| 1 |
| [[L-7](#low-7)] | `CultureIndex::createPiece()` and  creation of art piece with empty metadata| 1 |
| [[N-0](#Noncritical-0)] | Solidity function naming convention Not followed for external functions | 1 |
| [[N-1](#Noncritical-1)] | Solidity function naming convention Not followed for ineternal functions| 2 |
| [[N-2](#Noncritical-2)] | Contracts that are not mean't to be upgradeable should use normal inherited contracts| 2 |
| [[N-3](#Noncritical-3)] | Add Min and Max values for various setter function | 3 |


---

## Low Findings

### [L-0] Incorrect or Incomplete Natspac Comments <a id="low-0"></a>

Natspac is showing incorrect information:

_Instances: 1_

Function returns creator array length not total basis points.

```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/CultureIndex.sol

173     * @return Returns the total basis points calculated from the array of creators.
```
GitHub: [173](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L173)

_Instances: 2_

`timeBuffer` is the time that the contest end time will be increased by when the ending is in `timeBuffer` seconds. So the comment below is incorrect or showing incomplete info.

```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/AuctionHouse.sol

56      // The minimum amount of time left in an auction after a new bid is created
```
GitHub: [56](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L56)


_Instances: 3_

`MaxHeap::heap` is a mapping, not a struct.

```solidity

64    /// @notice Struct to represent an item in the heap by it's ID
65    mapping(uint256 => uint256) public heap;

```

Github: [64](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L64)

---

### [L-1] Creating a new Piece when ERC20 and ERC721 token supply is zero will cause issues <a id="low-1"></a>

`CultureIndex::createPiece(...)` allow creation of the peice even when `erc20VotingToken.totalSupply()` and `erc721VotingToken.totalSupply()` is zero will lead to creation of the peice with `totalVotesSupply` equal to zero which will lead to `quorumVotes` equal to zero. This will create the following issues:

1. Nobody will be able to vote on the tokens as the time of creation of the artpeice there was no token supply and because of this there will be no checkpoint for anyone in the ERC20 and ERC721 votes. And because of this `minVoteWeight` will not met.
2. As we know this `quorumVotes` will enable `AuctionHouse` to mint the tokens when this quorum is met. But if it is set to zero this will always enable `AuctionHouse` to mint the token even if nobody voted on the piece.

_instance: 1_

```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/CultureIndex.sol

226        newPiece.totalVotesSupply = _calculateVoteWeight(
227            erc20VotingToken.totalSupply(),
228            erc721VotingToken.totalSupply()
229        );
```


Github: [226-229](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L226-L229)

*_Recommendations:_*

Consider Implementing Initial Token Supply or move to redesign it.

---

### [L-2] Setting `Auction::reservePrice` equal to 0 can create a chain of 0 price bids <a id="low-2"></a>

After confirming from sponsor, it is allowed that `reservePrice` can be equal to zero. But this can create a chain of zero price bids and chances are that the auction may never stop as the time can increase by `timeBuffer` when time equal to or less `timeBuffer` is left for ending of the auction and new bid is raised. In order to correct that, there will be need to set `reservePrice` again to correct value. Here is test that proves that:

<details>
<summary>Code</summary>


```solidity
    function test_settingReservePriceEqualToZeroWillCreateChainOfZeroPriceBids() public {

            ////////////////////////////////////////////
            ///////      SETUP                   ///////
            ////////////////////////////////////////////

            address alice = makeAddr("alice");
            address bob = makeAddr("bob");
            address rachel = makeAddr("rachel");

            address[] memory tokenRecievers = new address[](1);
            tokenRecievers[0] = alice;

            uint256[] memory tokenSplitBps = new uint256[](1);
            tokenSplitBps[0] = 10000;

            IERC20TokenEmitter.ProtocolRewardAddresses memory protocolRewardsRecipients = IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            });

            ///////////////////////////////////////////////
            ////      Giving some ETH to users      ///////
            ///////////////////////////////////////////////

            vm.deal(alice, 100 ether);
            vm.deal(bob, 100 ether);

            //////////////////////////////////////
            //     Alice buying voting token  ////
            //////////////////////////////////////

            uint256 buyAmount = 10 ether;
            vm.startPrank(alice);
            erc20TokenEmitter.buyToken{value: buyAmount}(tokenRecievers, tokenSplitBps, protocolRewardsRecipients);
            vm.stopPrank();



            ///////////////////////////////////////
            ////    Creating New Token     ////////
            ///////////////////////////////////////

            uint256 artPieceId = createDefaultArtPiece();

            
            /////////////////////////////////////////////
            ///    Checking Created ArtPiece Info     ///
            /////////////////////////////////////////////

            ICultureIndex.ArtPiece memory artPeiceInfo = cultureIndex.getPieceById(artPieceId);

            assertEq(address(this), artPeiceInfo.sponsor, "sponsor is not correct");
            require(maxHeap.size() > 0, "MaxHeap is empty");


            //////////////////////////
            ///   Moving Blocks  /////
            //////////////////////////

            vm.roll(block.number + 1);


            ////////////////////////////////////////////////////////////////////////
            ////      Alice Saw the new peice And call vote function to vote.   ////
            ////////////////////////////////////////////////////////////////////////

            vm.prank(alice);
            cultureIndex.vote(artPieceId);


            ///////////////////////////////////////////
            ////    starting auction for the piece ////
            ///////////////////////////////////////////

            vm.startPrank(auction.owner());
            auction.unpause();

            //////////////////////////////////////////////
            ////    Setting reservePrice to zero      ////
            //////////////////////////////////////////////

            vm.startPrank(auction.owner());
            auction.setReservePrice(0);

            /////////////////////////////////////////////////////////////
            ////        bob bids on the artpeice for 0 eth          /////
            /////////////////////////////////////////////////////////////

            vm.startPrank(bob);
            auction.createBid{value: 0}(artPieceId, bob);    

            ////////////////////////////////////////////////////////////////
            ////        rachel bids on the artpeice for 0 eth          /////
            ////////////////////////////////////////////////////////////////


            vm.startPrank(rachel);
            auction.createBid{value: 0}(artPieceId, rachel);

            ///////////////////////////////////////////////////////////////////
            ////        bob again bids on the artpeice for 0 eth          /////
            ///////////////////////////////////////////////////////////////////

            vm.startPrank(bob);
            auction.createBid{value: 0}(artPieceId, bob);

    }

```


```solidity
    function createDefaultArtPiece() public returns (uint256) {
        return
            createArtPiece(
                "Mona Lisa",
                "A masterpiece",
                ICultureIndex.MediaType.IMAGE,
                "ipfs://legends",
                "",
                "",
                address(0x1),
                10000
            );
    }
```

</details>


_Recommendations:_

Consider Adding min And max limit for the `reservePrice` and never let it set equal to zero.

---

### [L-3] `Auction::_settleAuction()` can cause DoS if the remaining amount sent to `ERCTokenEmitter::buyToken()` is less than `RewardsSplit::minPurchaseAmount` <a id="low-3"></a>

The function `Auction::_settleAuction()` settles the current running auction by transferring the amounts to creators and other parties. But if the winning bid amount is less than `RewardsSplit::minPurchaseAmount` or `creatorBPS`(including entropy) is big enough that leaves only amount less than `RewardsSplit::minPurchaseAmount`, then the `_settleAuction` function will revert as the `buyToken` will not work because `RewardsSplit::computeTotalReward()` will revert if the amount is less than this min amount. The `settleAuction` will not be work and settling the auction will not be possible. The only way to solve this issue will be to raise `reservePrice` more than the current bid that will just refund the winning amount to deployer and will burn the current NFT.

Here is test that proves that: 

<details>
<summary>Code</summary>


```solidity
    function test_settlingAuctionWillNotBePossibleIfTheAmountSentToBuyTokenIsLessThanLimit() public {

            ////////////////////////////////////////////
            ///////      SETUP                   ///////
            ////////////////////////////////////////////

            address alice = makeAddr("alice");
            address bob = makeAddr("bob");
            address rachel = makeAddr("rachel");

            address[] memory tokenRecievers = new address[](1);
            tokenRecievers[0] = alice;

            uint256[] memory tokenSplitBps = new uint256[](1);
            tokenSplitBps[0] = 10000;

            IERC20TokenEmitter.ProtocolRewardAddresses memory protocolRewardsRecipients = IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            });

            ///////////////////////////////////////////////
            ////      Giving some ETH to users      ///////
            ///////////////////////////////////////////////

            vm.deal(alice, 100 ether);
            vm.deal(bob, 100 ether);

            //////////////////////////////////////
            //     Alice buying voting token  ////
            //////////////////////////////////////

            uint256 buyAmount = 10 ether;
            vm.startPrank(alice);
            erc20TokenEmitter.buyToken{value: buyAmount}(tokenRecievers, tokenSplitBps, protocolRewardsRecipients);
            vm.stopPrank();



            ///////////////////////////////////////
            ////    Creating New Token     ////////
            ///////////////////////////////////////

            uint256 artPieceId = createDefaultArtPiece();

            
            /////////////////////////////////////////////
            ///    Checking Created ArtPiece Info     ///
            /////////////////////////////////////////////

            ICultureIndex.ArtPiece memory artPeiceInfo = cultureIndex.getPieceById(artPieceId);

            assertEq(address(this), artPeiceInfo.sponsor, "sponsor is not correct");
            require(maxHeap.size() > 0, "MaxHeap is empty");


            //////////////////////////
            ///   Moving Blocks  /////
            //////////////////////////

            vm.roll(block.number + 1);


            ////////////////////////////////////////////////////////////////////////
            ////      Alice Saw the new peice And call vote function to vote.   ////
            ////////////////////////////////////////////////////////////////////////

            vm.prank(alice);
            cultureIndex.vote(artPieceId);


            ///////////////////////////////////////////
            ////    starting auction for the piece ////
            ///////////////////////////////////////////

            vm.startPrank(auction.owner());
            auction.unpause();

            ///////////////////////////////////////////////////
            ////    Setting reservePrice to low value      ////
            ///////////////////////////////////////////////////

            vm.startPrank(auction.owner());
            auction.setReservePrice(1000000);

            /////////////////////////////////////////////////////////////
            ////        bob bids on the artpeice for 0 eth          /////
            /////////////////////////////////////////////////////////////

            vm.startPrank(bob);
            auction.createBid{value: 0.00000001 ether}(artPieceId, bob);    

            ////////////////////////////////////
            ////    Moving Ahead in time    ////
            ////////////////////////////////////

            vm.warp(block.timestamp + 1 days);

            /////////////////////////////////////
            ////    Settling the auction    /////
            /////////////////////////////////////

            vm.expectRevert();
            auction.settleCurrentAndCreateNewAuction();
            
    }
```



```solidity
    function createDefaultArtPiece() public returns (uint256) {
        return
            createArtPiece(
                "Mona Lisa",
                "A masterpiece",
                ICultureIndex.MediaType.IMAGE,
                "ipfs://legends",
                "",
                "",
                address(0x1),
                10000
            );
    }
```


</details>


_Recommendations:_

Add a limit for the `reservePrice` keeping in mind the `RewardsSplit::minPurchaseAmount`. Make sure that value sent to `ERC20TokenEmitter:_handleRewardsAndGetValueToSend()` in `ERC20TokenEmitter::buyToken()` is never less than this min purchase amount.
Determining this value is kind of hard. If not possible than try to approach the whole thing in different way

---

### [L-4] `ERC20TokenEmitter` allow for buying token by passing deployer, builder and purchase refferal to his own address <a id="low-4">

A buyer can pass his own address as a deployer, builder and purchase refferal to get back his `2.5` percent ethers back. This is allowed for the external protocol integration but a normal user can take benefits from it. For external protocol integration, consider implementing this in a different way.

```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/ERC20TokenEmitter.sol

    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
@>        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {

        ...

        emit PurchaseFinalized(
            msg.sender,
            msg.value,
            toPayTreasury,
            msg.value - msgValueRemaining,
            uint256(totalTokensForBuyers),
            uint256(totalTokensForCreators),
            creatorDirectPayment
        );


        return uint256(totalTokensForBuyers);
    }
```

GitHub: [152-230](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L152C1-L230C6)

---

[L-5] `MaxHeap` is inheriting from `ReentrancyGuardUpgradeable`, but non reentrant check is not done anywhere in the contract. <a id="low-5"></a>

contract `MaxHeap` is inheriting from `ReentrancyGuardUpgradeable` but `nonReentrant` modifier is not used anywhere in the contract. Although the functions from the contract can be called by the admin only but it is good idea to add non reentrant modifier to be sure of reentrancy attacks. If not required then do not inherit from it.

_Instances: 1_

```solidity
File: packages/revolution/src/MaxHeap.sol

14. contract MaxHeap is VersionedContract, UUPS, Ownable2StepUpgradeable, ReentrancyGuardUpgradeable {

```

Github: [14](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L14)


---

[L-6] `MaxHeap::insert()` does not check if the value already exists for particular id and overwrites it <a id="low-6"></a>

`MaxHeap::insert()` does not check if the value already exists or not and overrides the old value. consider adding checks for that

```solidity
    function insert(uint256 itemId, uint256 value) public onlyAdmin {
        heap[size] = itemId;
        valueMapping[itemId] = value; // Update the value mapping
        positionMapping[itemId] = size; // Update the position mapping


        uint256 current = size;
        while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
            swap(current, parent(current));
            current = parent(current);
        }
        size++;
    }
```

GitHub: [119-130](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L119C1-L130C6)

---


### [L-7] `CultureIndex::createPiece()` and  creation of art piece with empty metadata <a id="low-7"></a>

New pieces can be created without giving any info for the metadata as there is no check in the function for the same. This could lead to creation of invalid tokens. Also `validateMediaType` only check for 3 types that allows creation of other tokens without having any of the info.

Here is a test that proves that:

```solidity

    function test_createAPieceWithoutAnymetadata() public {
            createArtPiece(
                "",
                "",
                ICultureIndex.MediaType.OTHER,
                "",
                "",
                "",
                address(0x1),
                10000
            );
    }

```

_Recommendations:_
Add better checks for NFT metadata.


---

## Non Critical Findings

### [N-0] Solidity function naming convention Not followed for external functions <a id="Noncritical-0" ></a>

According to solidity naming convetions, internal function name should start with `_`. But it is also used for an external function which might create confusion.

_Instance: 1_

```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/CultureIndex.sol


498    function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external onlyOwner {
```

Github: [498](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L498)

---

### [N-1] Solidity function naming convention Not followed for internal functions <a id="Noncritical-1" ></a>

According to solidity naming convetions, internal function name should start with `_`. But it is not used for some internal functions.

_Instance: 2_

```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/CultureIndex.sol


159     function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
```

Github: [159](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L159)


```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/CultureIndex.sol


179    function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
```

Github: [179](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L179)


---

[N-2] Contracts that are not mean't to be upgradeable should use normal inherited contracts <a id="Noncritical-2"></a>

Contracts like `NontransferableERC20Votes` and `ERC20TokenEmitter` are not inheriting from `UUPS`. Sponsor confirmed that these contracts should be non upgradeable. If that is the case then other inherited upgradeable contracts like `ReentrancyGuardUpgradeable`, `Ownable2StepUpgradeable` etc. should not be used to prevent unnecessary complexity and UX.

---

[N-3] Add Min and Max values for various setter function <a id="Noncritical-3"></a>

Giving infinte limit to variables might cause calculation and rounding issues. consider implementing the bounding limits.

_Instances AuctionHouse.sol: 3_

GitHub: [277](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L277)

GitHub: [287](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L287)

GitHub: [297](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L297)
