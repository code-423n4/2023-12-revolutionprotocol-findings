| ID | Title                                        | Severity   |
|----|----------------------------------------------|------------|
| [L01](#l01-metadata-not-properly-validated) | Metadata not properly validated          | Low Risk   |
| [L02](#l02-immutable-variables-not-preserved-between-upgrades) | Immutable variables not preserved between upgrades | Low Risk   |
| [L03](#l03-auction-parameters-can-be-changed-during-an-auction) | Auction parameters can be changed during an auction | Low Risk   |
| [L04](#l04-revert-state-in-auction-due-to-computetotalreward-function-failure) | Revert state in auction due to computeTotalReward function failure | Low Risk   |


## **L01**: Metadata not properly validated

## **Severity:**

- Low Risk

## **Relevant GitHub Links:**

- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L209
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L159
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L78

## **Summary:**

The CultureIndex contract permits the creation of new art pieces through the **createPiece** function. These pieces are subject to voting and potential minting based on the highest votes received. The **createPiece** function sets forth certain requirements for the art piece's metadata:

Requirements: 

- metadata must include name, description, and image. Animation URL is optional.
- creatorArray must not contain any zero addresses.
- The sum of basis points in creatorArray must be exactly 10,000. */

However, the **createPiece** function currently lacks verification for the presence of a name and description in the metadata fields.

```solidity

    function createPiece(ArtPieceMetadata calldata metadata, CreatorBps[] calldata creatorArray)
        public
        returns (uint256)
    {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

        // Validate the media type and associated data
        validateMediaType(metadata);
        ...
    }
```

Additionally, the **validateMediaType** function is called within **createPiece** to ensure requirements are met depending on the MediaType chosen. However, it does not cover all cases. A user can select from the following types:

```solidity
// Enum representing different media types for art pieces.
    enum MediaType {
        NONE,
        IMAGE,
        ANIMATION,
        AUDIO,
        TEXT,
        OTHER
    }
```

While the IMAGE, ANIMATION, and TEXT fields are checked to ensure a valid value is provided, the AUDIO type, which should fall under ANIMATION, is not checked.

- OpenSea Metadata Standards: https://docs.opensea.io/docs/metadata-standards

```solidity
/**
     * Requirements:
     * - The media type must be one of the defined types in the MediaType enum.
     * - The corresponding media data must not be empty.
     */
    function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

        if (metadata.mediaType == MediaType.IMAGE) {
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        } else if (metadata.mediaType == MediaType.ANIMATION) {
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        } else if (metadata.mediaType == MediaType.TEXT) {
            require(bytes(metadata.text).length > 0, "Text must be provided");
        }
    }
```

As a result, it is currently possible for art pieces to be created with missing fields in the metadata. Furthermore, as these pieces are auctioned off, an art piece that doesn’t follow the protocol's requirements could inadvertently be auctioned off for a substantial amount.

## **Impact:**

- Severity: Medium.  It is possible for art pieces with incomplete metadata to be created and auctioned.
- Likelihood: Low.  It is unlikely that an art piece with missing metadata would receive enough votes to be minted.

## **Tools Used:**

- Manual analysis

## **Recommendation:**

Add the following requirements to ensure the metadata requirements are followed by users when creating art pieces

```jsx
    function createPiece(ArtPieceMetadata calldata metadata, CreatorBps[] calldata creatorArray)
        public
        returns (uint256)
    {
        require(bytes(metadata.name).length > 0, "name must be provided"); //add here
        require(bytes(metadata.description).length > 0, "description must be provided"); //add here
        ...
    }
```

```solidity

    function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

        if (metadata.mediaType == MediaType.IMAGE) {
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        } else if (metadata.mediaType == MediaType.ANIMATION || metadata.mediaType == MediaType.AUDIO) { //add here
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        } else if (metadata.mediaType == MediaType.TEXT) {
            require(bytes(metadata.text).length > 0, "Text must be provided");
        }
    }
```
## **L02**: Immutable variables not preserved between upgrades

## **Severity:**

- Low Risk

## **Relevant GitHub Links:**

- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L85
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L85
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/Descriptor.sol#L60
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L23
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L109

## **Summary:**

The protocol employs the following upgradeable contracts: CultureIndex, MaxHeap, Descriptor, AuctionHouse, and VerbsToken. Despite these contracts being upgradeable they still initialize immutable variables through constructors. 

This approach is not recommended for proxy contracts because immutable variables do not persist through upgrades. If the implementation contract is upgraded without inheriting the first version or integrating the immutable variables into the new implementation, they will not be available in the new version, unlike storage variables that reside in the proxy contract.

```solidity
/// @notice The contract upgrade manager
    IRevolutionBuilder public immutable manager;

    ///                                                          ///
    ///                         CONSTRUCTOR                      ///
    ///                                                          ///

    /// @param _manager The contract upgrade manager address
    constructor(address _manager) payable initializer {
        manager = IRevolutionBuilder(_manager);
    }
```

## **Impact:**

This design can lead to unexpected behaviour in the upgraded contract versions, if the immutable values set in the original contract is not carried over.

## **Tools Used:**

- Manual analysis

## **Recommendation:**

Avoid using constructors for setting crucial contract parameters, instead use the initializer function to set necessary values.

## **L03**: Auction parameters can be changed during an auction

## **Severity:**

- Low Risk

## **Relevant GitHub Links:**

- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L277
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L287
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L297
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L336

## **Summary:**

The AuctionHouse contract is responsible for minting top-voted VerbsTokens and initiating auctions for them. Participants can bid on these auctions, and the highest bidder receives the art piece. The auction proceeds are then divided between the creators of the art piece and the owner of the auction contract.

```solidity
function settleCurrentAndCreateNewAuction() external override nonReentrant whenNotPaused {
        _settleAuction();
        _createAuction();
    }
```

An auction is concluded by invoking the **settleCurrentAndCreateNewAuction** function after the condition “block.timestamp >= _auction.endTime” is met. The **_settleAuction** function is responsible for transferring the art piece and distributing payments if all conditions are fulfilled. One such condition is that the contract's balance must be greater than the reserve price, signifying that a valid bid has been placed.

```solidity
function _settleAuction() internal {
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
            ...
    }
```

However, as it stands the contract owner can change the reserve price at any time using the **setReservePrice** function. This allows them to potentially block the highest bidder from winning by setting the reserve price higher than the contract's balance, leading to the burning of the piece and refunding the bidder.

```solidity
function setReservePrice(uint256 _reservePrice) external override onlyOwner {
        reservePrice = _reservePrice;

        emit AuctionReservePriceUpdated(_reservePrice);
    }
```

## **Impact:**

**Severity**: Low. While this ability to change the reserve price mid-auction could be misused to unfairly influence auction outcomes, its impact is limited given the owner address is a trusted role.

## **Tools Used:**

- Manual analysis

## **Recommendation:**

To enhance security, the trusted address in the contract, particularly for sensitive functions like setting auction parameters, should be managed through a multi-signature wallet. This approach ensures that important decisions require consensus among multiple trusted parties, reducing the risk of unauthorized changes due to a compromised account.

## **L04**: Revert state in auction due to computeTotalReward function failure

## **Severity:**

- Low Risk

## **Relevant GitHub Links:**

- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L336
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40

## **Summary:**

The **_settleAuction** function in the AuctionHouse contract is responsible for finalizing auctions, transferring art pieces to the highest bidder, and paying out the creator and owner. The function has implemented a fail safe method by pausing the contract when the _settleAuction function reverts. 

## **Vulnerability Details:**

The auction is concluded using the **settleCurrentAndCreateNewAuction** function, which is triggered after the condition “**block.timestamp >= _auction.endTime**” is met. Within this process, the **_settleAuction** function handles the transfer of the art piece and the distribution of payments.

```solidity
function _settleAuction() internal {
        ...
                //Buy token from ERC20TokenEmitter for all the creators
                if (creatorsShare > ethPaidToCreators) {
                    creatorTokensEmitted = erc20TokenEmitter.buyToken{value: creatorsShare - ethPaidToCreators}(
                        vrgdaReceivers,
                        vrgdaSplits,
                        IERC20TokenEmitter.ProtocolRewardAddresses({
                            builder: address(0),
                            purchaseReferral: address(0),
                            deployer: deployer
			...
    }
```

The **buyToken** function is internally called, it purchases tokens for creators and the buyer. This function includes a step where 2.5% of the purchase is allocated as rewards, calculated by the **computeTotalReward** function.

```solidity
function buyToken(
        ...
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        ...
        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );
        ...
    }
```

This function is designed with specific constraints: if the **paymentAmountWei** falls below the minimum threshold (**0.0000001 ether**) or exceeds the maximum limit (**50,000 ether**), the function will trigger a revert. This can lead to potential issues in certain transaction scenarios.

```solidity
function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
				...
    }
```

While a direct call to **`buyToken`** simply reverts the transaction, a call through **`_settleAuction`** can lead to the auction being locked until the auction's split percentages are adjusted or the tokens are burnt.

## **Impact:**

**Locking of Auctions**: This issue can result in auctions being locked and unable to proceed, requiring manual intervention to resolve

## **Tools Used:**

- Manual analysis

## **Recommendation:**

Implement a fallback mechanism in the **`_settleAuction`** function to handle amounts outside the desired range.