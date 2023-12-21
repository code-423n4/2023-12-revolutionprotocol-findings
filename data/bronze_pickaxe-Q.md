# L-01 `validateMediaType` does not validate audio

The team states in the comments above the function `validateMediaType`:
```javascript
    /**
     *  Validates the media type and associated data.
     * @param metadata The metadata associated with the art piece.
     *
     * Requirements:
     * - The media type must be one of the defined types in the MediaType enum.
     * - The corresponding media data must not be empty.
     */
```
`corresponding media data must not be empty`.

It checks if the data is empty in the function `validateMediaType`:
```javascript
        if (metadata.mediaType == MediaType.IMAGE)
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        else if (metadata.mediaType == MediaType.ANIMATION)
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        else if (metadata.mediaType == MediaType.TEXT)
            require(bytes(metadata.text).length > 0, "Text must be provided");
```

However, there is no check for `MediaType.AUDIO` and no check for `MediaType.OTHER`.

**Recommendation**: add these checks to the `validateMediaType` function:
```diff
        if (metadata.mediaType == MediaType.IMAGE)
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        else if (metadata.mediaType == MediaType.ANIMATION)
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        else if (metadata.mediaType == MediaType.TEXT)
            require(bytes(metadata.text).length > 0, "Text must be provided");
+	    else if (metadata.mediaType == MediaType.AUDIO)
+            require(bytes(metadata.text).length > 0, "Data must be provided");
+	    else if (metadata.mediaType == MediaType.OTHER)
+            require(bytes(metadata.text).length > 0, "Data must be provided");
```


