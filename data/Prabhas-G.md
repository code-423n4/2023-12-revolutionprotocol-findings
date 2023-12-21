# We can optimize the `maxHeapify()` function: saves 207 gas from the tests included:
[https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L94-#L113)

|        | Min | Average | Median | Max |
|---------|-----|---------|--------|-----|
| Before  | 13248 | 13248   | 13248  | 13248 |
| After   | 13041 | 13041   | 13041  | 13041 |

```solidity
    function maxHeapify(uint256 pos, uint256 magnitude) public {
        uint256 left = 2 * pos + 1;
        uint256 right = 2 * pos + 2;

        uint256 posValue = valueMapping[heap[pos]];
        uint256 leftValue = valueMapping[heap[left]];
        uint256 rightValue = valueMapping[heap[right]];


        if (pos >= (size / 2) && pos <= size) return;

        if (posValue < leftValue || posValue < rightValue) {
            if (leftValue > rightValue) {
                swap(pos, left);
                maxHeapify(left);
            } else {
                swap(pos, right);
                maxHeapify(right);
            }
        }
    }
```
The above function reads the state variable size 2*total number of times called which is quite expensive as gas cost for SLOAD operations are high.

As this function is recursive storing the `size` state variable don't have much effect as if loads from storage on every function call. we can reduce this gas cost by passing the `size` as function parameter from the calling function. 
```
function maxHeapify(uint256 pos, uint256 _size) public {
    uint256 left = 2 * pos + 1;
    uint256 right = 2 * pos + 2;

    uint256 posValue = valueMapping[heap[pos]];
    uint256 leftValue = valueMapping[heap[left]];
    uint256 rightValue = valueMapping[heap[right]];


    if (pos >= (_size / 2) && pos <= _size)return;

    if (posValue < leftValue || posValue < rightValue) {
        if (leftValue > rightValue) {
            swap(pos, left);
            maxHeapify(left, _size);
        } else {
            swap(pos, right);
            maxHeapify(right, _size);
        }
   }
}

function maxHeapifyHelper(uint256 pos) public {
	uint magnitude = size;
	maxHeapify(pos, magnitude);
		
}
```
But this additionally requires creating a helper function and calling it wherever the `maxHeapify()` function is called. 


# The state variable `verbs` is read multiple times in `_settleAuction()` function of `AuctionHouse.sol` contract.
In the following function the state variable `verbs` is read more than one.
```
File: packages/revolution/src/AuctionHouse.sol

  function _settleAuction() internal {
        IAuctionHouse.Auction memory _auction = auction;

			.			.			.			.
			.			.			.			.
			.			.			.			.

       

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

						.			.			.			.
						.			.			.			.
						.			.			.			.
```

Multiple `SLOAD` operations can be prevented by caching the value of `verbs` state variable value in `memory`.

```
  function _settleAuction() internal {
        IAuctionHouse.Auction memory _auction = auction;
		IVerbsToken memory _verbs = verbs;

			.			.			.			.
			.			.			.			.
			.			.			.			.

       

        uint256 creatorTokensEmitted = 0;
        // Check if contract balance is greater than reserve price
        if (address(this).balance < reservePrice) {
            // If contract balance is less than reserve price, refund to the last bidder
            if (_auction.bidder != address(0)) {
                _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
            }

            // And then burn the Noun
            _verbs.burn(_auction.verbId);
        } else {
            //If no one has bid, burn the Verb
            if (_auction.bidder == address(0))
                _verbs.burn(_auction.verbId);
                //If someone has bid, transfer the Verb to the winning bidder
            else _verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

            if (_auction.amount > 0) {
                // Ether going to owner of the auction
                uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;

                //Total amount of ether going to creator
                uint256 creatorsShare = _auction.amount - auctioneerPayment;

                uint256 numCreators = _verbs.getArtPieceById(_auction.verbId).creators.length;
                address deployer = _verbs.getArtPieceById(_auction.verbId).sponsor;

                //Build arrays for erc20TokenEmitter.buyToken
                uint256[] memory vrgdaSplits = new uint256[](numCreators);
                address[] memory vrgdaReceivers = new address[](numCreators);

                //Transfer auction amount to the DAO treasury
                _safeTransferETHWithFallback(owner(), auctioneerPayment);

                uint256 ethPaidToCreators = 0;

                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                if (creatorsShare > 0 && entropyRateBps > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = _verbs.getArtPieceById(_auction.verbId).creators[i];
                        vrgdaReceivers[i] = creator.creator;
                        vrgdaSplits[i] = creator.bps;

                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
                    }
                }

						.			.			.			.
						.			.			.			.
						.			.			.			.
```

# The state variable `creatorsAddress` should be cached in `memory` to prevent multiple SLOAD operations.

```
File: packages/revolution/src/ERC20TokenEmitter.sol

function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        //prevent treasury from paying itself
        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

        require(msg.value > 0, "Must send ether");
        // ensure the same number of addresses and bps
        require(addresses.length == basisPointSplits.length, "Parallel arrays required");

        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );

        //Share of purchase amount to send to treasury
        uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;

        //Share of purchase amount to reserve for creators
        //Ether directly sent to creators
        uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
        //Tokens to emit to creators
        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
            : int(0);

        // Tokens to emit to buyers
        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);

        //Transfer ETH to treasury and update emitted
        emittedTokenWad += totalTokensForBuyers;
        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

        //Deposit funds to treasury
        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
        require(success, "Transfer failed.");

        //Transfer ETH to creators
        if (creatorDirectPayment > 0) {
            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed.");
        }

        //Mint tokens for creators
        if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
            _mint(creatorsAddress, uint256(totalTokensForCreators));
        }

 .		.		.		.
 .		.		.		.
 .		.		.		.

```

As we can see in the above function the state variable `creatorsAddress` is read multiple times.

```
 .		.		.		.
 .		.		.		.
 .		.		.		.
require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

 .		.		.		.
 .		.		.		.
 .		.		.		.

if (creatorDirectPayment > 0) {
	(success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
	require(success, "Transfer failed.");
}

//Mint tokens for creators
if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
	_mint(creatorsAddress, uint256(totalTokensForCreators));
}

.		.		.		.
.		.		.		.
.		.		.		.
```
This variable should be cached in memory to save from multiple SLOAD operations.


# Multiple `SSTORE` operations on `emittedTokenWad` can be prevented by caching the intermediate result and storing the final result in one sstore operation.

```
File: packages/revolution/src/ERC20TokenEmitter.sol

function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        //prevent treasury from paying itself
        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

        require(msg.value > 0, "Must send ether");
        // ensure the same number of addresses and bps
        require(addresses.length == basisPointSplits.length, "Parallel arrays required");

        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );

        //Share of purchase amount to send to treasury
        uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;

        //Share of purchase amount to reserve for creators
        //Ether directly sent to creators
        uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
        //Tokens to emit to creators
        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
            : int(0);

        // Tokens to emit to buyers
        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);

        //Transfer ETH to treasury and update emitted
        emittedTokenWad += totalTokensForBuyers;
        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;



 .		.		.		.
 .		.		.		.
 .		.		.		.
```

The values of `totalTokensForBuyers` and `totalTokensForCreators` can be stored in single variable and the combined result can be stored in `emittedTokenWad`.


```
// store the value of totalTokensForBuyers in new initialized variable

uint totalTokens = totalTokensForBuyers;


// update the totalTokens if necessary

if (totalTokensForCreators > 0) totalTokens += totalTokensForCreators;


// Then store the combined result in single sstore operation

emittedTokenWad = totalTokens;

```