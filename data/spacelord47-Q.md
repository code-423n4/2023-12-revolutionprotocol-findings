## Summary

|ID|Issue|Instances|Severity|
|-|:-|:-:|-|
| [L&#x2011;01] | Incorrect param validation in `getArtPieceById` function | 1 | Low |
| [L&#x2011;02] | Lack of Verbs token id validation when fetching token URI | 2 | Low |
| [L&#x2011;03] | Missing data validation for art pieces added to CultureIndex | 1 | Low | 
| [N&#x2011;01] | Confusing comments | 1 | Non-Critical | 
| [N&#x2011;02] | Duplicate code logic | 1 | Non-Critical | 

---

## [L-01] Incorrect param validation in `getArtPieceById` function

`getArtPieceById` function in `VerbsToken` contract used to fetch a `ArtiPice` by id (`verbId` parameter). This function contains logic to validate `verbId` at line 274.

[src/VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L273-L275)
```solidity
273: function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
274:     require(verbId <= _currentVerbId, "Invalid piece ID");
275:     return artPieces[verbId];
276: }
```

The problem here is that `verbId == _currentVerbId` is not a valid case, because `_currentVerbId` holds id for the next token to be minted (and not an existing id), as we can observe on the following lines:

[src/VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L58)
```solidity
58: // The internal verb ID tracker
59: uint256 private _currentVerbId;

// @audit Updating id after minting new art piece using post-increment
220: uint256 verbId = _currentVerbId++;

```

`CultureIndex` contract contains proper validation for similar logic:
[src/CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L308)
```solidity
307: function _vote(uint256 pieceId, address voter) internal {
308:     require(pieceId < _currentPieceId, "Invalid piece ID");
```

### Recommendation

Update the condition to exclude invalid case

```diff
diff --git a/packages/revolution/src/VerbsToken.sol b/packages/revolution/src/VerbsToken.sol
--- a/packages/revolution/src/VerbsToken.sol	(revision d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6)
+++ b/packages/revolution/src/VerbsToken.sol	(date 1703185529843)
@@ -271,7 +271,7 @@
      * @return The ArtPiece struct associated with the given ID.
      */
     function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
-        require(verbId <= _currentVerbId, "Invalid piece ID");
+        require(verbId < _currentVerbId, "Invalid piece ID");
         return artPieces[verbId];
     }
```

## [L-02] Lack of Verbs token id validation when fetching token URI

`VerbsToken` contracts implements [ERC-721 standard](https://eips.ethereum.org/EIPS/eip-721) and its optional "metadata" extension.
According to the standard, `tokenURI` function *"Throws if `_tokenId` is not a valid NFT"*. But the implementation violates this requirement  because doesn't validate `tokenId` parameter:

[src/VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/VerbsToken.sol#L193-L195)
```solidity
193: function tokenURI(uint256 tokenId) public view override returns (string memory) {
194:     return descriptor.tokenURI(tokenId, artPieces[tokenId].metadata);
195: }
```

If we follow the `descriptor.tokenURI` call, we can see that there is also no validation on `tokenId`:

[src/Descriptor.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/ce98201adbc538dd52a24c9677fc238ae8ef09a2/packages/revolution/src/Descriptor.sol#L144-L151)
```solidity
144: function tokenURI(
145:     uint256 tokenId,
146:     ICultureIndex.ArtPieceMetadata memory metadata
147: ) external view returns (string memory) {
148:     if (isDataURIEnabled) return dataURI(tokenId, metadata);
149: 
150:     return string(abi.encodePacked(baseURI, tokenId.toString()));
151: }
```

Also, there is `dataURI` function with similar logic, which also missing validation for `tokenId` (even though it's not a part of ERC-721):

[src/VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/VerbsToken.sol#L201-L203)
```solidity
201:  function dataURI(uint256 tokenId) public view override returns (string memory) {
202:      return descriptor.dataURI(tokenId, artPieces[tokenId].metadata);
203:  }
```

Missing `tokenId` validation might result in end users receiving (invalid) data, if non-existent (or malformed) token id provided.

### Impact

If non-existent (or malformed) `tokenId` provided, invalid data will be returned by the function, which might negatively impact end users experience.

### Recommendation

Use `_requireOwned` helper function from [ERC721Upgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/6dfb5c683528ed5de06a125e9215d10de8909b3b/contracts/token/ERC721/ERC721Upgradeable.sol#L477-L483) to validate `tokenId` :

```diff
diff --git a/packages/revolution/src/VerbsToken.sol b/packages/revolution/src/VerbsToken.sol
--- a/packages/revolution/src/VerbsToken.sol	(revision d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6)
+++ b/packages/revolution/src/VerbsToken.sol	(date 1703185596929)
@@ -191,6 +191,7 @@
      * @dev See {IERC721Metadata-tokenURI}.
      */
     function tokenURI(uint256 tokenId) public view override returns (string memory) {
+        _requireOwned(tokenId);
         return descriptor.tokenURI(tokenId, artPieces[tokenId].metadata);
     }
 
@@ -199,6 +200,7 @@
      * with the JSON contents directly inlined.
      */
     function dataURI(uint256 tokenId) public view override returns (string memory) {
+        _requireOwned(tokenId);
         return descriptor.dataURI(tokenId, artPieces[tokenId].metadata);
     }
```

## [L-03] Missing data validation for art pieces added to CultureIndex

`CultureIndex.createPiece` is a public function and used by creators to add new art pieces to the index.
The issue here is that the `metadata` parameter lacks of validation for it's `name` and `desription` fields (which is stated at line 205):

[src/CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/CultureIndex.sol#L195-L248)
```solidity
195:     /**
196:      * @notice Creates a new piece of art with associated metadata and creators.
197:      * @param metadata The metadata associated with the art piece, including name, description, image, and optional animation URL.
198:      * @param creatorArray An array of creators who contributed to the piece, along with their respective basis points that must sum up to 10,000.
199:      * @return Returns the unique ID of the newly created art piece.
200:      *
201:      * Emits a {PieceCreated} event for the newly created piece.
202:      * Emits a {PieceCreatorAdded} event for each creator added to the piece.
203:      *
204:      * Requirements:
205:      * - `metadata` must include name, description, and image. Animation URL is optional.
206:      * - `creatorArray` must not contain any zero addresses.
207:      * - The sum of basis points in `creatorArray` must be exactly 10,000.
208:      */
209:     function createPiece(
210:         ArtPieceMetadata calldata metadata,
211:         CreatorBps[] calldata creatorArray
212:     ) public returns (uint256) {
213:         uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
214: 
215:         // Validate the media type and associated data
216:         validateMediaType(metadata); // @audit This function only partially validates metadata
217: 
218:         uint256 pieceId = _currentPieceId++;
219: 
220:         /// @dev Insert the new piece into the max heap
221:         maxHeap.insert(pieceId, 0);
222: 
223:         ArtPiece storage newPiece = pieces[pieceId];
224: 
225:         newPiece.pieceId = pieceId;
226:         newPiece.totalVotesSupply = _calculateVoteWeight(
227:             erc20VotingToken.totalSupply(),
228:             erc721VotingToken.totalSupply()
229:         );
230:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
231:         newPiece.metadata = metadata;
232:         newPiece.sponsor = msg.sender;
233:         newPiece.creationBlock = block.number;
234:         newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
235: 
236:         for (uint i; i < creatorArrayLength; i++) {
237:             newPiece.creators.push(creatorArray[i]);
238:         }
239: 
240:         emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
241: 
242:         // Emit an event for each creator
243:         for (uint i; i < creatorArrayLength; i++) {
244:             emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
245:         }
246: 
247:         return newPiece.pieceId;
248:     }
```

There are some validation inside `validateMediaType` function, but it doesn't validate `name` and `description` fields:

[src/CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/CultureIndex.sol#L159-L168)
```solidity
159:     function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
160:         require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");
161: 
162:         if (metadata.mediaType == MediaType.IMAGE)
163:             require(bytes(metadata.image).length > 0, "Image URL must be provided");
164:         else if (metadata.mediaType == MediaType.ANIMATION)
165:             require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
166:         else if (metadata.mediaType == MediaType.TEXT)
167:             require(bytes(metadata.text).length > 0, "Text must be provided");
168:     }
```

### Impact

This lack of validation will allow users to create art pieces with invalid metadata and potentially polluting the index with obsolete entries.

### Recommendation

Rename `validateMediaType` to `validatePieceMetadata` and add basic validation for `name` and `description`

```diff
diff --git a/packages/revolution/src/CultureIndex.sol b/packages/revolution/src/CultureIndex.sol
--- a/packages/revolution/src/CultureIndex.sol	(revision d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6)
+++ b/packages/revolution/src/CultureIndex.sol	(date 1703172327703)
@@ -156,8 +156,10 @@
      * - The media type must be one of the defined types in the MediaType enum.
      * - The corresponding media data must not be empty.
      */
-    function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
+    function validatePieceMetadata(ArtPieceMetadata calldata metadata) internal pure {
         require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");
+        require(bytes(metadata.name).length > 0, "Name must be provided");
+        require(bytes(metadata.description).length > 0, "Description must be provided");
 
         if (metadata.mediaType == MediaType.IMAGE)
             require(bytes(metadata.image).length > 0, "Image URL must be provided");
@@ -213,7 +215,7 @@
         uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
 
         // Validate the media type and associated data
-        validateMediaType(metadata);
+        validatePieceMetadata(metadata);
 
         uint256 pieceId = _currentPieceId++;
```

---

## [N-01] Confusing comments

1. Remove comment or update "@notice" to match "only admin" logic.

[src/MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/MaxHeap.sol#L38-L44)
```solidity
 38:     /**
 39:      * @notice Require that the minter has not been locked.
 40:      */
 41:     modifier onlyAdmin() {
 42:         require(msg.sender == admin, "Sender is not the admin");
 43:         _;
 44:     }
```

## [N-02] Duplicate code logic

1. `VerbsToken._mintTo` internal function responsible for minting new Verbs tokens, The function performs validation on `artPiece.creators.length`  fetched from `CultureIndex`:
[src/VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/VerbsToken.sol#L281-L289)
```solidity
281:     function _mintTo(address to) internal returns (uint256) {
282:         ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();
283: 
284:         // Check-Effects-Interactions Pattern
285:         // Perform all checks
286:         require(
287:             artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
288:             "Creator array must not be > MAX_NUM_CREATORS"
289:         );
```

but the validation here is not necessary because we validate creators array when registering new art piece in `CultureIndex` (line 182):

[src/CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/CultureIndex.sol#L179-L182)
```solidity
179:     function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
180:         uint256 creatorArrayLength = creatorArray.length;
181:         //Require that creatorArray is not more than MAX_NUM_CREATORS to prevent gas limit issues
182:         require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

...

209:     function createPiece(
210:         ArtPieceMetadata calldata metadata,
211:         CreatorBps[] calldata creatorArray
212:     ) public returns (uint256) {
213:         uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
```

Having duplicate logic complicates code maintainability and makes it less readable.

### Recommendation

Remove duplicated logic from `VerbsToken._mintTo`:

```diff
diff --git a/packages/revolution/src/VerbsToken.sol b/packages/revolution/src/VerbsToken.sol
--- a/packages/revolution/src/VerbsToken.sol	(revision d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6)
+++ b/packages/revolution/src/VerbsToken.sol	(date 1703179960510)
@@ -281,13 +281,6 @@
     function _mintTo(address to) internal returns (uint256) {
         ICultureIndex.ArtPiece memory artPiece = cultureIndex.getTopVotedPiece();
 
-        // Check-Effects-Interactions Pattern
-        // Perform all checks
-        require(
-            artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
-            "Creator array must not be > MAX_NUM_CREATORS"
-        );
-
         // Use try/catch to handle potential failure
         try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
             artPiece = _artPiece;
```
