## General
1. ### Contracts can't be deployed with the default solidity version on optimism and base chains.

Considering that the the contracts are expected to be deployed on mmainnet, optimism ad base chains, the solidity version that should be in use must be suported by all the chains.
However, the contracts in the protocol are using compiler version 0.8.22 which compiles normally on Ethereum mainnet. However, optimism and base chains while EVM-compatible, do not support the PUSH0 opcode which was introduced in the Shanghai hard fork, which is now the default EVM version in the compiler and the one being currently used to compile the project. The 0.8.22 compiler version produces bytecode that contains the PUSH0 opcode. This will result in a failure when trying to deploy the contracts to Optimism and Base. Consider using Solidity compiler version 0.8.19.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L2

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L2

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/NontransferableERC20Votes.sol#L4

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L2

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L24

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L18

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/libs/VRGDAC.sol#L2

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L2

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L2

***
2. ### Auction process can be put to stop if the `ERC20tokenEmitter` is paused

The AuctionHouse depends on the `ERC20tokenEmitter` contract to buy the votes token. The `_settleAuction` function is executes the `buyToken` call in the `ERC20TokenEmitter` which can only be called when the emitter is not paused. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L400
```
function _settleAuction() internal { 
                  ...
                //Buy token from ERC20TokenEmitter for all the creators
                if (creatorsShare > ethPaidToCreators) {
                    creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: creatorsShare - ethPaidToCreators }(
                  ...
        }
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L152
```
    function buyToken(
        address[] calldata addresses, //vgrda rewards
        uint[] calldata basisPointSplits, //percents
        ProtocolRewardAddresses calldata protocolRewardsRecipients 
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) //@note
```
[Pausing](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L132) this contract puts a halt to a core part of the protocol's functionality in that auctions cannot be settled, and consequently, new auctions cannot be created. 

Recommend reconsidering the property, probably disabling it in the `ERC20TokenEmitter`, or allowing the `AuctionHouse` to override it.

***
***

## [Cultureindex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol)

1. ### Metadata requirements are not properly enforced
The [`validateMediaType`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L159) function is used to validate a media type and its data. From the provided comment,
```
     * Requirements:
     * - The media type must be one of the defined types in the MediaType enum.
     * - The corresponding media data must not be empty.
      
```
However, the function only validates certain media types and specific characteristics. 
```
        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

        if (metadata.mediaType == MediaType.IMAGE)
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        else if (metadata.mediaType == MediaType.ANIMATION)
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        else if (metadata.mediaType == MediaType.TEXT)
            require(bytes(metadata.text).length > 0, "Text must be provided");
```
even though there are five possible [media types](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L78) that can be uploaded.

```
    enum MediaType {
        NONE, //@note not needed
        IMAGE,
        ANIMATION,
        AUDIO,
        TEXT,
        OTHER
    }
```
and six distinct [metadata](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L88) characteristics.
```
    struct ArtPieceMetadata {
        string name;
        string description;
        MediaType mediaType;
        string image;
        string text;
        string animationUrl;
    }
```
Due to lax validation in the `validateMediaType` function, `Other` and `Audio` pieces can be created with only the media type data(lacking other possibly important params), and generally, Image, Animation and Text data can be created without names or description. 
As it stands, pieces can be uploaded without name, description, possibly image, text, animationuri depending on its type. These parameters are used to [construct](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/Descriptor.sol#L97C1-L111C102) the piece's `tokenUri` when it eventually gets minted as a verb token. Not having these parameters gives the tokens (Other and Audio) an empty `tokenUri`. The lax validation also leaves the tokenUri vulnerable to JSON injection.
```
    function constructTokenURI(TokenURIParams memory params) public pure returns (string memory) {
        string memory json = string(
            abi.encodePacked(
                '{"name":"',
                params.name,
                '", "description":"',
                params.description,
                '", "image": "',
                params.image,
                '", "animation_url": "',
                params.animation_url,
                '"}'
            )
        );
        return string(abi.encodePacked("data:application/json;base64,", Base64.encode(bytes(json))));
```

Consider improving on this. 

***
2. ### Lack of vetoer increases the risk of 51% attack 

The veto power provides a mechanism for a member or group within the DAO to reject or override a proposal. The helps to protect from malicious hijacking by users. There is no veto power in the [`Cultureindex`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol) contract, which opens up to 51% attack. Malicious or unwanted artpieces can be upvoted by a select few who manage to hijack more than half of the total voting weight. 
Consider implementing a veto function

***
***

## [Verbstoken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol)

1. ### Consider checking for zero addresses before locking the `Descriptor` and `CultureIndex` addresses

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L242C1-L246C6

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L262
```
    /**
     * @notice Lock the descriptor.
     * @dev This cannot be reversed and is only callable by the owner when not locked.
     */
    function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {
        isDescriptorLocked = true;

        emit DescriptorLocked();
    }
    ...
    /**
     * @notice Lock the CultureIndex
     * @dev This cannot be reversed and is only callable by the owner when not locked.
     */
    function lockCultureIndex() external override onlyOwner whenCultureIndexNotLocked {
        isCultureIndexLocked = true;

        emit CultureIndexLocked();
    }
 ```
As there are no 0 address checks when setting the CultureIndex and Descriptor addresses both in the [initializer](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L154C1-L155C53) and the [setter](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L230C1-L256C6) functions, consider implementing a check for 0 address or that they have been set before locking. This is important as there's no way to update these addresses after they're locked except redeployment.
 
***
***



## [Auctionhouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol)

1. ### Consider capping the `creatorRateBps` to < 100% (also applies to the same function in the [`ERC20TokenEmitter`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol) contract)

The [`setCreatorRateBps`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L217) allows for setting `creatorRateBps` so high that the auction owner and treasury doesn't get paid dues. 
```
    function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
        require(
            _creatorRateBps >= minCreatorRateBps, //@note
            "Creator rate must be greater than or equal to minCreatorRateBps"
        );
        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000"); //@note
        creatorRateBps = _creatorRateBps;

        emit CreatorRateBpsUpdated(_creatorRateBps);
    }
```
This same also goes for [`_minCreatorRateBps`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L233) which is can be set so high, the `creatorBps` has to be adjusted so that it can be match the minimum needed value. This also can lead to a point where the auction owners do not get paid any dues. Important to note that this change cannot be [reversed](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L238C1-L241C11) because the `setMinCreatorRateBps` function requires the new `minCreatorRateBps` be always greater than old value.
```
 require(
            _minCreatorRateBps > minCreatorRateBps,
            "Min creator rate must be greater than previous minCreatorRateBps"
        );
```
This also applies to the [`_entropyRateBps`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L253), it can be set so high that auction creators no longer get paid any votes tokens.

Recommend considering if this is desired property or capping to `< 100%` instead, that way the relevant parties can still get paid.

***
***
