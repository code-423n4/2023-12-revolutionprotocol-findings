### L-01: Function **CultureIndex.createPiece** has no way of validating duplicates art pieces

Function **createPiece** is currently implemented as follows:

```solidity
function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

        // Validate the media type and associated data
        validateMediaType(metadata);

        uint256 pieceId = _currentPieceId++;
....
```
As you can see, only the creators array and media type is checked, no duplication check. This could lead to an art piece is sold for more than once on Revolution Protocol


### L-02: AuctionHouse's owner should not be allowed to change parameters when auction is active

**AuctionHouse** contract contains config values such as *timeBuffer*, *reservePrice*:
```solidity
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

```
All of these params could be changed during an active auction is running, as you can see for example function *setReservePrice* has no restriction:

```solidity
unction setReservePrice(uint256 _reservePrice) external override onlyOwner {
        reservePrice = _reservePrice;

        emit AuctionReservePriceUpdated(_reservePrice);
    }

```
The owner should be forced to settle the current auction, pause the contract and then update these params.
