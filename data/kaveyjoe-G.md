## G - 01  - Bitwise Arithmetic for Efficient Calculations

Use bitwise operations for arithmetic calculations. The left shift (<<) operation can replace multiplication by powers of 2, which is more gas-efficient.

**Original Code:**

    function maxHeapify(uint256 pos) internal {
        uint256 left = 2 * pos + 1;
        uint256 right = 2 * pos + 2;
        // ... rest of the function ...
    }
**Optimized Code:**

    function maxHeapify(uint256 pos) internal {
        uint256 left = (pos << 1) + 1;
        uint256 right = left + 1;
        // ... rest of the function ...
    }

## G -02  Direct State Variable Updates to Reduce Gas

 Minimize storage reads and writes. Instead of creating a local storage pointer, directly update the storage variable, which costs less gas.

Original Code:

    function vote(uint256 artPieceId, uint256 amount) external {
        require(amount > 0, "Cannot vote with 0 amount");
        ArtPiece storage artPiece = artPieces[artPieceId];
        artPiece.voteCount += amount;
        // ... rest of the function ...
    }
Optimized Code:

    function vote(uint256 artPieceId, uint256 amount) external {
        require(amount > 0, "Cannot vote with 0 amount");
        artPieces[artPieceId].voteCount += amount;
        // ... rest of the function ...
    }


## G -03  Preventing Zero-Value Transactions for Gas Savings

Add require checks to prevent unnecessary state changes. By ensuring that the mint function is not called with a zero address or amount, we can save gas by avoiding needless transactions.

Original Code:

    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
        // ... rest of the function ...
    }
Optimized Code:

    function mint(address to, uint256 amount) external onlyOwner {
        require(to != address(0), "Mint to the zero address");
        require(amount != 0, "Mint 0 amount");
        _mint(to, amount);
        // ... rest of the function ...
    }


## G -04  Ensuring Meaningful Interactions to Avoid Wasted Gas

Ensure meaningful interactions. By adding a check to ensure that the token amount is greater than zero, we prevent users from spending gas on transactions that would have no effect.

Original Code:

    function buyToken() external payable {
        uint256 tokenAmount = calculateTokenAmount(msg.value);
        _mint(msg.sender, tokenAmount);
        // ... rest of the function ...
    }
Optimized Code:

    function buyToken() external payable {
        uint256 tokenAmount = calculateTokenAmount(msg.value);
        require(tokenAmount > 0, "Token amount is zero");
        _mint(msg.sender, tokenAmount);
        // ... rest of the function ...
    }

## G -05   Caching State Variables for Fewer SLOAD Operations

Cache frequently accessed state variables. By storing auctionEnd in a local variable, we reduce the cost of multiple reads from the same state variable.

Original Code:

    function settleAuction() external {
        require(auctionEnd < block.timestamp, "Auction not yet ended");
        // ... rest of the function ...
    }
Optimized Code:

    function settleAuction() external {
        uint256 _auctionEnd = auctionEnd;
        require(_auctionEnd < block.timestamp, "Auction not yet ended");
        // ... rest of the function ...
    }


## G -06  Optimizing State Changes Order to Save Gas on Reverts

Delay state changes until necessary. By incrementing the _tokenIdCounter after the minting operation, we can save gas if the minting fails and the transaction reverts.

Original Code:

    function mintArtPiece(address to) external onlyOwner {
        uint256 tokenId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        _mint(to, tokenId);
        // ... rest of the function ...
    }
Optimized Code:

    function mintArtPiece(address to) external onlyOwner {
        uint256 tokenId = _tokenIdCounter.current();
        _mint(to, tokenId);
        _tokenIdCounter.increment();
        // ... rest of the function ...
    }
