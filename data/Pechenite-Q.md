## [L-01] Protocol logic not following comments
Above `CultureIndex.sol#createPiece()` there is a comment that states

```solidity
     * - `metadata` must include name, description, and image. Animation URL is optional.
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L205C81-L205C81)

the following function calls `validateMediaType()` which on its own contains check that will revert if an Animation URL is not provided.

```solidity
159:    function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
160:        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");
        ...
164:        else if (metadata.mediaType == MediaType.ANIMATION)
165:            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        ...
168:    }
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L165C21-L165C21)

---

## [L-02] Possible rounding issues

Inside the `CultureIndex.sol#createPiece()` function, the following code is used to calculate the required `quorumVotes` for piece to be selected:

```solidity
234:    newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L234)

There are certain values of `quorumVotesBPS` and `newPiece.totalVotesSupply` that when multiplied are lower than 10,000 will cause the result of this calculation to be rounded down to 0 due to Solidity not supporting fractions. For example, if `quorumVotesBPS` is 90 and `newPiece.totalVotesSupply` is 100, the result of the calculation will be 0. This will cause the piece to be selected without quorum votes

---

## [L-03] Some mutable state variables cannot be changed

Inside the `CultureIndex.sol` contract there are two state variables `minVoteWeight` and `erc721VotingTokenWeight` that cannot be changed in any way. This could break the future flow of the protocol if the values of these variables need to be changed. Consider adding setters for these variables.

```solidity
45:     uint256 public erc721VotingTokenWeight;
...
51:     uint256 public minVoteWeight;
```
[45](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L45), [51](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L51C29-L51C29)

---

## [L-04] Internal function `_authorizeUpgrade()` in some contracts is not used

The following instances of `_authorizeUpgrade()` are not used internally, consider removing them.

```solidity
File: CultureIndex.sol
543:    function _authorizeUpgrade(address _newImpl) internal view override onlyOwner {
544:        // Ensure the new implementation is a registered upgrade
545:        if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
546:    }
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L543)

```solidity
File: AuctionHouse.sol
452:    function _authorizeUpgrade(address _newImpl) internal view override onlyOwner {
453:        // Ensure the new implementation is a registered upgrade
454:        if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
455:    }
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L452)

```solidity
File: Descriptor.sol
187:    function _authorizeUpgrade(address _impl) internal view override onlyOwner {
188:        if (!manager.isRegisteredUpgrade(_getImplementation(), _impl)) revert INVALID_UPGRADE(_impl);
189:    }
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/Descriptor.sol#L187)

```solidity
File: MaxHeap.sol
181:  function _authorizeUpgrade(address _newImpl) internal view override onlyOwner {
182:        // Ensure the new implementation is a registered upgrade
183:        if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
184:    }
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L181)

```solidity
File: VerbsToken.sol
328:    function _authorizeUpgrade(address _newImpl) internal view override onlyOwner {
329:        // Ensure the implementation is valid
330:        require(manager.isRegisteredUpgrade(_getImplementation(), _newImpl), "Invalid upgrade");
331:    }
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L328)

---
