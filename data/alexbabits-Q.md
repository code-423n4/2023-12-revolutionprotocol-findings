## `[Low-1]`

* **Description**: Require statement that is a double negative could cause confusion inside `CultureIndex::_vote()`.
* **Location**: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L311
* **Original Line**: `require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");`. "It must be not true that the voter is not the 0 address". This will pass the check anytime the voter address is 0x0.
* **Recommendation**: `require((votes[pieceId][voter].voterAddress = address(0)), "Already voted");`. "The voter address must be the 0 address". This will pass the check anytime the voter address is 0x0. Otherwise we know it has already voted because the address value in the mapping defaults to 0 when the voter address has not voted.

## `[Low-2]`

* **Description**: Incorrect comment on inequality logic.
* **Location**: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L347-#L348
* **Recommendation**: 
```diff
-       // Check if contract balance is greater than reserve price
+       // Check if contract balance is less than the reserve price
        if (address(this).balance < reservePrice) {
```

## `[Low-3]`

* **Description**: State variable `quorumVotesBPS` can be set to 0 during initialization and in it's setter, causing the number of quorum votes to create a new piece for an auction to be 0.
* **Locations**: 
	* Initialization: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L119
	* `CreatePiece()` impact: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L234
	* `_setQuorumVotesBPS()` setter: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L498-#L503
* **Recommendation**: Consider having a `!=0` or `MIN_QUORUM_VOTES_BPS` as a "fat-finger" safety check when initializing `quorumVotesBPS`, and/or for the `CultureIndex::_setQuorumVotesBPS()` function.


## `[Low-4]`

* **Description**: The owner can change `minBidIncrementPercentage` state variable to 0. This causes `AuctionHouse::createBid()` to allow multiple bidders to place the same bid amount. 
	* Allowing multiple people to bid the same amount doesn't make practical sense.  
	* This could also result in a griefing DoS for the current auction. Imagine the market price of an art piece is around 2 ETH. Alex and Bob are malicious whales that could bid back and forth at 4 ETH with the bid amount never increasing, until Charlie bids a higher amount (unlikely, it's already double the market value) or the `minBidIncrementPercentage` or `timeBuffer` is manually changed by the owner. 
	* It would cost the whales only gas per bid, and the loss of (4-2 = 2) ETH whenever they decide to end the bid and win the item. The `timeBuffer` is said to be 15 minutes, so it would only cost about 100 calls to keep the auction going for 24 hours.
* **Location**: 
	* Setter: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L297-#L301
	* `createBid()` consequences: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L180-#L183
* **Recommendation**: Consider having a `!=0` or `MIN_BID_INCREMENT_PERCENTAGE` as a "fat-finger" safety check when calling `AuctionHouse::setMinBidIncrementPercentage()`.
