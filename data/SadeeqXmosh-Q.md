## L-01: function `validateMediaType` doesnot validate other 3 types . 
function `validateMediaType` is for validating the data provided by user . However , validation for  3 types of piece  `( NONE,AUDIO, OTHER )` is missing . function `validateMediaType` only validates Image , Animation & Text type . 
    
       
```solidity 
 function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

        if (metadata.mediaType == MediaType.IMAGE)
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        else if (metadata.mediaType == MediaType.ANIMATION)
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        else if (metadata.mediaType == MediaType.TEXT)
            require(bytes(metadata.text).length > 0,

```

Code link: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L159

### Recommendation:

Implement necessary validation for other 3 types . 

## L-02: `creatorArray[i].bps` should not be 0 ! 
`validateCreatorsArray` function is for validating ` creatorArray` param . However it should be checked that no one of the creators have a bps of 0 . There's no point having creator with bps . It will increase gas usage , storage issues & potential DOS . 
```solidity 
  function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
  //.....
 for (uint i; i < creatorArrayLength; i++) {
            require(creatorArray[i].creator != address(0), "Invalid creator address");
            totalBps += creatorArray[i].bps; //@audit creatorArray[i].bps should not be 0 ! 
        }

```

Code link:https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L187C45-L187C45

### Recommendation:
Check for `creatorArray[i].bps != 0` : 
```diff 
 for (uint i; i < creatorArrayLength; i++) {
            require(creatorArray[i].creator != address(0), "Invalid creator address");
+           require(creatorArray[i].bps != 0, "Invalid creator bps");
            totalBps += creatorArray[i].bps; //@audit creatorArray[i].bps should not be 0 ! 
        }
```


