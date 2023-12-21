## QA report



#### [Q-1] Setter function do not have limitation of value.
It is clear that the onlyOwner role is a trusted role, but it will be more convenient and better for users if there is some maximum value, beyond which MinBidIncrementPercentage cannot be set.
```diff
    function setMinBidIncrementPercentage(uint8 _minBidIncrementPercentage) external override onlyOwner {
+       require(_minBidIncrementPercentage <= MAXBidIncrementPercentage, "Too big value");
		minBidIncrementPercentage = _minBidIncrementPercentage;
		
        emit AuctionMinBidIncrementPercentageUpdated(_minBidIncrementPercentage);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L297-L301

#### [Q-2] Owner could change important auction parameters during its active phase.
Changes to parameters should be made only when the auction is paused and completely completed, and a new one has not yet begun. Need add whenPaused modifier

```diff
-   function setReservePrice(uint256 _reservePrice) external override onlyOwner {
+   function setReservePrice(uint256 _reservePrice) external override whenPaused onlyOwner {
+       IAuctionHouse.Auction memory _auction = auction;
+       require(_auction.settled, "Auction has not been settled");
        ...
    }

-   function setEntropyRateBps(uint256 _entropyRateBps) external onlyOwner {
+       IAuctionHouse.Auction memory _auction = auction;
+       require(_auction.settled, "Auction has not been settled");
    ...
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L287-L290
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L253-L257

#### [Q-3] Value of minimal gas should not be constant
Writing the exact gas value into a constant is bad practice. There have already been several cases when the price of opcodes has changed upward. It is recommended to make a function with the ability to change this value
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L88

#### [Q-4] Unreachable code
During normal use of a contract, there cannot be a situation where the contract balance is equal to or greater than reservePrice. Therefore this code will never be executed. But the miner will indicate the address of the auction contract to receive a reward for creating a block, then yes, a situation will arise that the contract balance is greater than or equal to reservePrice and if no one has made a bid. The willow token will be burned. But the airwaves will remain under contract. You need to either remove this section of code, or add sending ether to the owner of the contract.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L358-L360

#### [Q-5] There is not special fields audio/other types of media in ArtPieceMetadata struct
The validateMediaType function checks that the creator has filled out the field with the address to the media object. But only for some types of media. For audio, for example, there is no such check. Because in the metadata structure there is no corresponding field to fill in the address that would point to the address of the audio file. The creator will only be able to specify the name and description of the audio media

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L159-L168

#### [Q-6] The error does not give an exact answer as to which signature is incorrect
Signatures are checked in a for cycle, and if one of the signatures is incorrect, then a single custom error is used for all this. The user does not understand which signature is incorrect. It is necessary to display the serial number of the incorrect signature in the error
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L404

#### [Q-7] Owner could not change address of dropperAdmin
There is not function in contract, which allow owner change dropperAdmin address. Owner could set value for this variable only while initialize contract. Variable is not immutable. Recommended add setter with modifier onlyOwner for changing address of dropper admin. 
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L78

#### [Q-8] The same error for 2 different transfers
The function has 2 messages that have the same reverse message. The user will not be able to understand which transfer did not work. 
```solidity
    (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
        require(success, "Transfer failed."); // <---------------
      
        //Transfer ETH to creators
        if (creatorDirectPayment > 0) {
            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed."); // <---------------
        }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L192-L197

#### [Q-9] There not need nonReentrant modifier in function with modifier onlyOwner
Because owner is trusted role. 
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L309

#### [Q-10] Owner can set the value of _creatorRateBps directly to the function initial to avoid unnecessary calls.
The initial value can be set in the initialize function. This has already been done for the creatorsAddress variable

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L299-L303

#### [Q-11] No need to read the returned value
The WETH contract always returns TRUE at the end of the transfer() function. If any error occurs, then the revert triggers. Therefore, we do not need to read and check the returned value.
```solidity
// WETH contract from ethereum mainnet - https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2

    function transfer(address dst, uint wad) public returns (bool) {
        return transferFrom(msg.sender, dst, wad);
    }

    function transferFrom(address src, address dst, uint wad)
        public
        returns (bool)
    {
        require(balanceOf[src] >= wad);

        if (src != msg.sender && allowance[src][msg.sender] != uint(-1)) {
            require(allowance[src][msg.sender] >= wad);
            allowance[src][msg.sender] -= wad;
        }

        balanceOf[src] -= wad;
        balanceOf[dst] += wad;

        Transfer(src, dst, wad);

        return true;  // <--------------------
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L435-L438

#### [Q-12] User can not cancel his vote
The user cannot cancel his vote; usually this function is available until a quorum is reached in other projects.
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L332-L334

#### [Q-13] Owner could not change admin address in MaxHeap contract
The admin address is set only in the initialization function. There is not function for change it.
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L58

#### [Q-14] Unused import ReentrancyGuard
None of the functions in this file contain a nonReentrant modifier, so you do not need to import and initialize the ReentrancyGuardUpgradeable file
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L5