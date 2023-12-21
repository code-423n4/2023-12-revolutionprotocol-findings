## GAS report



#### [G-1] Unnecessary verification of incoming amount
This is not a necessary check because then the computeTotalReward function is still called with the same parameter.
_depositPurchaseRewards(msg.value,..) -> computePurchaseRewards() -> computeTotalReward()
```diff
  function _handleRewardsAndGetValueToSend(
        uint256 msgValue,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {
-       if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();

        return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18

#### [G-2] It is better to move the verification of bidder is address(0) (rare case) further from the beginning of the function
We can save the user gas by moving the verification of a rare case further from the beginning of the function. In order for another check to work - which is more common, the user did not spend gas on a rare check
​
```diff
  function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
        IAuctionHouse.Auction memory _auction = auction;

        //require bidder is valid address
-       require(bidder != address(0), "Bidder cannot be zero address");
        require(_auction.verbId == verbId, "Verb not up for auction");
        //slither-disable-next-line timestamp
        require(block.timestamp < _auction.endTime, "Auction expired");
        require(msg.value >= reservePrice, "Must send at least reservePrice");
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );
+       require(bidder != address(0), "Bidder cannot be zero address");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L175

#### [G-3] Combine 2 IF in 1
We can transfer emit function to the first if
```diff
 function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
    ...
-   if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
+   if (extended) {
+       auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
+       emit AuctionExtended(_auction.verbId, _auction.endTime);
+   }

    // Refund the last bidder, if applicable
    if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);

    emit AuctionBid(_auction.verbId, bidder, msg.sender, msg.value, extended);

-   if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L192-L199

#### [G-4] Use block.timestamp directly
No need to read the variable from the memory, because the value of the block.timestamp has not changed and we can use it directly
```diff
 uint256 startTime = block.timestamp;
 auction = Auction({
                verbId: verbId,
                amount: 0,
-               startTime: startTime, 
+               startTime: block.timestamp, 
                endTime: endTime,
                bidder: payable(0),
                settled: false
            });

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L320

#### [G-5] Delete the unattainable code and combine two IF
There cannot be a situation where the balance of the contract is greater than the value of the variable Reserveprice and at the same time bidder is a zero address. Therefore, this branching must be removed and united with lower IF
```diff
 function _settleAuction() internal {
    ...
   if (address(this).balance < reservePrice) {
            // If contract balance is less than reserve price, refund to the last bidder
            if (_auction.bidder != address(0)) {
                _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
            }

            // And then burn the Noun
            verbs.burn(_auction.verbId);
        } else {
-           //If no one has bid, burn the Verb
-            if (_auction.bidder == address(0))
-               verbs.burn(_auction.verbId);
-               //If someone has bid, transfer the Verb to the winning bidder
-            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

-            if (_auction.amount > 0) {
                // Ether going to owner of the auction
                uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L348C2-L363

#### [G-6] No need check return value of weth.transfer() function
This value always - TRUE. If was any error while transfering, it will revert.
```diff
 function _safeTransferETHWithFallback(address _to, uint256 _amount) private {
    ...
    
    IWETH(WETH).deposit{ value: _amount }();

    // Transfer WETH instead
-   bool wethSuccess = IWETH(WETH).transfer(_to, _amount);
+   IWETH(WETH).transfer(_to, _amount);

-   // Ensure successful transfer
-   if (!wethSuccess) revert("WETH transfer failed");
```

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

#### [G-7] Combine 2 cycles in 1
Two For loops pass along the same array. We can unite them in 1
```diff
 function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
    ...
    for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
+           emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
    }

    emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
-   // Emit an event for each creator
-   for (uint i; i < creatorArrayLength; i++) {
-       emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
-   }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L236-L245


#### [G-8] Unnecessary check for zero address
It is not necessary to check the value of the FROM variable at the zero address, because even if this is, then there is a check that the recoveredadaddress does not equal to the zero address and is not even equal to the address From
```diff
function _verifyVoteSignature(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal returns (bool success) {
    ...
-     // Ensure to address is not 0
-     if (from == address(0)) revert ADDRESS_ZERO();

      // Ensure signature is valid
     if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L437-L441

#### [G-9] Unnecessary converting to bytes
There is no need to convert 0 into bytes and deliver it as a date of tx.
```diff
-  (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
+  (bool success, ) = treasury.call{ value: toPayTreasury }(hex"0");
+  // OR even
+  (bool success, ) = treasury.call{ value: toPayTreasury }("");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L191
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L196

#### [G-10] Unnecessary check for zero address
There is no need to check here at the zero address, because the address of creators has already been checked  when was creating Piece. CultureIndex.sol.createPiece() -> validateCreatorsArray()
```solidity
function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
        uint256 creatorArrayLength = creatorArray.length;
        //Require that creatorArray is not more than MAX_NUM_CREATORS to prevent gas limit issues
        require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

        uint256 totalBps;
        for (uint i; i < creatorArrayLength; i++) {
            require(creatorArray[i].creator != address(0), "Invalid creator address"); // <------
            totalBps += creatorArray[i].bps;
        }
```
```diff
function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
    ...
    //Mint tokens for creators
-   if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
+   if (totalTokensForCreators > 0) {
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L201

#### [G-11] Unnecessary modifier nonReentrant for onlyOwner function
Owner is a trusted person who will not do malicious actions.
```diff
-  function setCreatorsAddress(address _creatorsAddress) external override onlyOwner nonReentrant {
+  function setCreatorsAddress(address _creatorsAddress) external override onlyOwner {
        require(_creatorsAddress != address(0), "Invalid address");

        emit CreatorsAddressUpdated(creatorsAddress = _creatorsAddress);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L309

#### [G-12] Unnecessary default value of private variable
No need to set default value in the variable because this value will be rewritten in initializer function.
```solidity
 function initialize(
        address _minter,
        address _initialOwner,
        address _descriptor,
        address _cultureIndex,
        IRevolutionBuilder.ERC721TokenParams memory _erc721TokenParams
    ) external initializer {
    ...
    
     _contractURIHash = _erc721TokenParams.contractURIHash; // <------
```
```diff
-  string private _contractURIHash = "QmQzDwaZ7yQxHHs7sQQenJVB89riTSacSGcJRv9jtHPuz5";
+  string private _contractURIHash;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L63

#### [G-13] Double check of creators array length
No need to check the length of creators array for the second time because it has already been done when creating piece. CultureIndex.sol.createPiece() -> validateCreatorsArray(creatorArray)
```solidity
// CultureIndex.sol.validateCreatorsArray()
    function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
        uint256 creatorArrayLength = creatorArray.length;
        //Require that creatorArray is not more than MAX_NUM_CREATORS to prevent gas limit issues
        
        // HERE <------------------
        require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

```
```diff
   function _mintTo(address to) internal returns (uint256) {
        ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();

        // Check-Effects-Interactions Pattern
        // Perform all checks
-       require(
-           artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
-           "Creator array must not be > MAX_NUM_CREATORS"
-       );
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L286-L288

#### [G-14] Unnecessary reading value from storage
If we are in this block, then the function cultureindex.droptopvotedpiece() has been successfully completed and it always write True value for isDropped field, so we do not need to read this value, and we can immediately write the work
```solidity
    function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");

        ICultureIndex.ArtPiece memory piece = getTopVotedPiece();
        require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");

        //set the piece as dropped
        pieces[piece.pieceId].isDropped = true; // <----------

```
```diff
  try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
            artPiece = _artPiece;
            uint256 verbId = _currentVerbId++;

            ICultureIndex.ArtPiece storage newPiece = artPieces[verbId];

            newPiece.pieceId = artPiece.pieceId;
            newPiece.metadata = artPiece.metadata;
-           newPiece.isDropped = artPiece.isDropped; 
+           newPiece.isDropped = true;
            ...
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L300

#### [G-15] Double check size of heap size
No need check heap size in CultureIndex.topVotedPieceId(), because in thecks in MaxHeap.getMax()
```diff
// CultureIndex.topVotedPieceId()
   function topVotedPieceId() public view returns (uint256) {
-       require(maxHeap.size() > 0, "Culture index is empty");
        //slither-disable-next-line unused-return
        (uint256 pieceId, ) = maxHeap.getMax();
        return pieceId;
    }
// MaxHeap.getMax()
    function getMax() public view returns (uint256, uint256) {
        require(size > 0, "Heap is empty"); // <--------------
        return (heap[0], valueMapping[heap[0]]);
    }
````

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L487

#### [G-16] Unused import ReentrancyGuard
None of the functions in contract contain a nonReentrant modifier, so you do not need to import and initialize the ReentrancyGuardUpgradeable file
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L5

#### [G-17] unused block Else
In the project, the value always only increases, so we don’t need a block with code that decreases the value
```diff
   function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
       ...
        if (newValue > oldValue) {
            // Upwards heapify
            while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
                swap(position, parent(position));
                position = parent(position);
            }
          

          
-       } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify
+      } 
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L150

#### [G-18] Unused return value
The function returns values, but they are not read by the calling function
```diff
// CultureIndex.dropTopVotedPiece()
   function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");

        ICultureIndex.ArtPiece memory piece = getTopVotedPiece();
        require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");

        //set the piece as dropped
        pieces[piece.pieceId].isDropped = true;

        //slither-disable-next-line unused-return
        maxHeap.extractMax();  // <-----------

        emit PieceDropped(piece.pieceId, msg.sender);

        return pieces[piece.pieceId];
    }

// MaxHeap.extractMax()
-  function extractMax() external onlyAdmin returns (uint256, uint256) {
+ function extractMax() external onlyAdmin  {
        require(size > 0, "Heap is empty");

        uint256 popped = heap[0];
        heap[0] = heap[--size];
        maxHeapify(0);
        // @audit-issue [G] лишний ретурн - он не использутеся
-        return (popped, valueMapping[popped]);
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L163

#### [G-19] Unused second return value
```diff
// CultureIndex.topVotedPieceId()
  function topVotedPieceId() public view returns (uint256) {
        require(maxHeap.size() > 0, "Culture index is empty");
        //slither-disable-next-line unused-return
        (uint256 pieceId, ) = maxHeap.getMax(); // <--------
        return pieceId;
    }
// MaxHeap.extractMax()
-   function extractMax() external onlyAdmin returns (uint256, uint256) {
+  function extractMax() external onlyAdmin returns (uint256) {
        require(size > 0, "Heap is empty");

        uint256 popped = heap[0];
        heap[0] = heap[--size];
        maxHeapify(0);
      
-       return (popped, valueMapping[popped]);
+      return popped;
    }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L169-L172

