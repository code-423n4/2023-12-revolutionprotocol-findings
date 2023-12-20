# Low
## [L-01] `validateMediaType` does not check the metadata of AUDIO art piece.
The metadata of all art piece types are defined and validated except AUDIO type.
```solidity
File: packages/revolution/src/CultureIndex.sol

159:    function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
160:        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");
161:
162:        if (metadata.mediaType == MediaType.IMAGE)
163:            require(bytes(metadata.image).length > 0, "Image URL must be provided");
164:        else if (metadata.mediaType == MediaType.ANIMATION)
165:            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
166:        else if (metadata.mediaType == MediaType.TEXT)
167:            require(bytes(metadata.text).length > 0, "Text must be provided");  // @audit no checking for AUDIO type
168:    }
```
[CultureIndex.sol#L159-L168](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/CultureIndex.sol#L159-L168)
```solidity
File: packages/revolution/src/interfaces/ICultureIndex.sol

88:    struct ArtPieceMetadata {
89:        string name;
90:        string description;
91:        MediaType mediaType;
92:        string image;
93:        string text;
94:        string animationUrl;  // @audit no metadata for AUDIO type
95:    }
```
[ICultureIndex.sol#L88-L95](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/interfaces/ICultureIndex.sol#L88-L95)

## [L-02] Animation URL is mandatory, not optional.
The comment for metadata is incorrect (`L205`). The animation URL is mandatory and not optional according to [validateMediaType](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/CultureIndex.sol#L164-L165)(`L164-L165`).
```solidity
File: packages/revolution/src/CultureIndex.sol

164:        else if (metadata.mediaType == MediaType.ANIMATION)
165:            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");

205: * - `metadata` must include name, description, and image. Animation URL is optional.  // @audit incorrect comment
```
[CultureIndex.sol#L205](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/CultureIndex.sol#L205)

## [L-03] Incorrect comment for `newQuorumVotesBPS`.
The hardcoded quorum votes bps (`MAX_QUORUM_VOTES_BPS`) is max instead of min.
```solidity
File: packages/revolution/src/CultureIndex.sol

495:     * @dev newQuorumVotesBPS must be greater than the hardcoded min
```
[CultureIndex.sol#L495](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/CultureIndex.sol#L495)

## [L-04] Require the token is minted when querying metadata of the token.
First check if the token is minted before returning the token's metadata, avoiding return invalid data for non-existing token.
```solidity
File: packages/revolution/src/VerbsToken.sol

193:    function tokenURI(uint256 tokenId) public view override returns (string memory) {
194:        return descriptor.tokenURI(tokenId, artPieces[tokenId].metadata);   // @audit `_requireOwned(tokenId)`
195:    }

200:    function dataURI(uint256 tokenId) public view override returns (string memory) {
201:        return descriptor.dataURI(tokenId, artPieces[tokenId].metadata);    // @audit `_requireOwned(tokenId)`
202:    }
```
[VerbsToken.sol#L193-L195](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/VerbsToken.sol#L193-L195),[VerbsToken.sol#L200-L202](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/VerbsToken.sol#L200-L202)

## [L-05] Consider adding `unlock` functions for `minter`, `descriptor` and `cultureIndex`.
The lock functions (`lockMinter`, `lockDescriptor`, `lockCultureIndex`) are risky when no unlock functions are provided. Once locked, there's no way to unlock it, which means it is unable to upgrade `minter`, `descriptor` and `cultureIndex`.
```solidity
File: packages/revolution/src/VerbsToken.sol

220:    function lockMinter() external override onlyOwner whenMinterNotLocked
242:    function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {
262：   function lockCultureIndex() external override onlyOwner whenCultureIndexNotLocked {
```
[VerbsToken.sol#L220](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/VerbsToken.sol#L220),
[VerbsToken.sol#L242](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/VerbsToken.sol#L242),
[VerbsToken.sol#L262](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/VerbsToken.sol#L262)

## [L-06] Incorrect checking for auction `reservePrice`.
Use `_auction.amount < reservePrice` instead of `address(this).balance < reservePrice`, as `_auction.amount` is the real bid amount of the auction and it is possible that `address(this).balance != _auction.amount`.
```solidity
File: packages/revolution/src/AuctionHouse.sol

348:        if (address(this).balance < reservePrice) {  // @audit use `_auction.amount < reservePrice`
349:            // If contract balance is less than reserve price, refund to the last bidder
350:            if (_auction.bidder != address(0)) {
351:                _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
352:            }
```
[AuctionHouse.sol#L348-L352](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/AuctionHouse.sol#L348-L352)


# Non-Critical
## [N-01] Check creator's bps is larger than 0.
```solidity
File: packages/revolution/src/CultureIndex.sol

185:        for (uint i; i < creatorArrayLength; i++) {
186:            require(creatorArray[i].creator != address(0), "Invalid creator address");
187:            totalBps += creatorArray[i].bps; // @audit require `creatorArray[i].bps != 0`
188:        }
```
[CultureIndex.sol#L185-L188](https://github.com/code-423n4/2023-12-revolutionprotocol/tree/main/packages/revolution/src/CultureIndex.sol#L185-L188)