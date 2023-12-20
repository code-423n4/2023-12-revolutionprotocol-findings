in https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L78-L95

```
    enum MediaType {
        NONE,
        IMAGE,
        ANIMATION,
        AUDIO,
        TEXT,
        OTHER
    }

    // Struct defining metadata for an art piece.
    struct ArtPieceMetadata {
        string name;
        string description;
        MediaType mediaType;
        string image;
        string text;
        string animationUrl;
    }
```

ArtPieceMetada is the struct for holding metadata of an NFT where we have
- `string image` for the MediaType.IMAGE
- `string test` for the MediaType.TEXT
- `string animationUrl` for the MediaType.ANIMATION
but there is no metadata variable for holding data from MediaType.AUDIO and MediaType.Other