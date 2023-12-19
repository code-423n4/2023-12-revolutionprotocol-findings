## Finding Summary 

| ID | Description | Severity |
| - | - | :-: |
| [L-01](#l-01-unnecessary-abundance-of-roles) | `verbId` parameter is incorrectly checked in `getArtPieceById()` | Low |
| [L-02](#l-02-unchecked-return-data-when-implementing-multiexcall) | Reliance on `address(this).balance` in `AuctionHouse` | Low |
| [L-03](#l-03-users-have-no-way-of-revoking-their-signatures-in-lenshub) | `getPastVotes()` gives the unfair voting privileges to newly created art pieces | Low |
| [L-04](#l-04-removing-erc721enumerable-functionality-might-break-composability-with-other-protocols) | `OTHER` mediatype is initialized but it's not validated  | Low |
| [N-01](#n-01-approvefollow-should-allow-followerprofileid--0-to-cancel-follow-approvals) | `totalErc20Supply` parameter is not used anywhere | Non-Critical |
| [N-02](#n-02-redundant-check-in-_isapprovedorowner-can-be-removed) | `weight` should be `>=` than `minWeight` | Non-Critical |
| [N-03](#n-03-whennotpaused-modifier-is-redundant-for-burn-in-lensprofilessol) | Excessive checks for `creators.length` | Non-Critical |
| [N-04](#n-04-unfollow-should-be-allowed-for-burnt-profiles) |  Hard-coded gas limit can lead to tx revert in some cases  | Non-Critical |
| [N-05](#n-04-unfollow-should-be-allowed-for-burnt-profiles) | Inconsistent checks in `AuctionHouse` | Non-Critical |



## [L-01] `verbId` parameter is incorrectly checked in `getArtPieceById()`

Inside of `VerbsToken` contract there is a function that checks whether the `verbId` less or equal to `_currentVerbId`. However, it uses `<=` operator instead of `<` as it's done in the `CultureIndex` contract when checking for `pieceId`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L308
```
require(pieceId < _currentPieceId, "Invalid piece ID");
```

The problem is that the when `pieceId` and `verbId` are created, they are assigned to the id of the `_currentPieceId` and `_currentVerbId` accordingly. But the `currentVerbId` is increased by one when assigning and it cannot be equal to the `verbId` therefore:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L273-276

```
  function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
        require(verbId <= _currentVerbId, "Invalid piece ID");
        return artPieces[verbId];
    }
```

In the case of the first creation:

```
verbId = 0
_currentVerbId = 1
```

### Recommendation

Change `<=` on `<` when calling `getArtPieceById()`.


## [L-02] Reliance on `address(this).balance` allows other users to send tokens
and always bypass the check in `AuctionHouse` 

Inside of `AuctionHouse.sol` smart contract, there is a check that compares `address(this).balance` with `reservePrice`. And if the `reservePrice` is less than the balance, then the last bidder will be refunded and `verbId` will be burnt.

However, this assumption doesn't take into account the fact that the ETH can be sent to the smart contract directly via `selfdestruct()`, for example. So if the value of ETH sent equals to or becomes bigger than the `reservePrice` variable, this check will simply always bypass.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L348-352

 if (address(this).balance < reservePrice) {
            // If contract balance is less than reserve price, refund to the last bidder
            if (_auction.bidder != address(0)) {
                _safeTransferETHWithFallback(_auction.bidder, _auction.amount)
     }
    
### Recommendation

Change the check and remove the `address(this).balance` as parameter that is used to compare with `reservePrice` variable.


## [L-03] `getPastVotes()` gives the unfair voting privileges to newly created art pieces

According to the docs, the users can use their voting power to vote on different art pieces only if they had that voting power at the time of creation (`block.number`).

However, the problem is that this will disincentivize to create art pieces early and everybody would wait until somebody has already created their art pieces and enough number of voters are already present in the system. That could cause a situation where users just wait for each other action and don't create anything:


https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L313
```
  uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);
```

### Recommendation

Use `block.timestamp` to determine whether the user can vote.

## [L-04] `OTHER` mediatype is initialized but it's not validated 

Inside of `CultureIndex.sol` contract, `validateMediaType()` function is used only to validate `TEXT`, `VIDEO` and `ANIMATION` enum members but it also allows to create media type of type `OTHER`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L216
```
 validateMediaType(metadata);
```

`MediaType` enum:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/interfaces/ICultureIndex.sol#L78-85
```
  enum MediaType {
        NONE,
        IMAGE,
        ANIMATION,
        AUDIO,
        TEXT,
        OTHER
    }
```

This would allow other users to put `OTHER` as mediaType and provide `image` or `animationUrl` or `text` with zero length that is validated. Therefore, it can bypass the `validateMediatype()` check:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L159-168

```
function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
        require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");

        if (metadata.mediaType == MediaType.IMAGE)
            require(bytes(metadata.image).length > 0, "Image URL must be provided");
        else if (metadata.mediaType == MediaType.ANIMATION)
            require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
        else if (metadata.mediaType == MediaType.TEXT)
            require(bytes(metadata.text).length > 0, "Text must be provided");
    }
```

### Recommendation

Add the check for `OTHER` metadata type inside of `validateMediaType` and check for the length of all three (`metadata.image`, `metadata.animationUrl`, `metadata.text`) in this scenario or just remove this from the types.

## [N-01] `totalErc20Supply` parameter is not used anywhere

When `newPiece` is created inside of `CultureIndex`, it's not used anywhere after the initiatialization so it can be easily removed:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L230
```
 newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
```

### Recommendation

Remove `totalErc20Supply` param or use it somewhere in the logic.

## [N-02] `weight` should be `>=` than `minWeight`

In the current implementation, when checking whether the weight of the voter is bigger than the minimum weight, the following operator is used:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L314
```
require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");
```
### Recommendation

Change on `weight >= `minWeight`.

But the name of the `minVoteWeight` suggests that it's the minimum weight that can be used for voting and, therefore, it should be included in the check as well. Otherwise, it remains confusing.

## [N-03] Excessive checks for `creators.length`

`creators.length` parameter is checked twice both in `CultureIndex` and `VerbsToken`:

`CultureIndex.sol`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L213

```
   uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
``` 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L182
```
require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");
```

So this check would not allow to create art piece with bigger number of creators than `MAX_NUM_CREATORS` at first place.

`VerbsToken.sol`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L286-288
```
  require(
            artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
            "Creator array must not be > MAX_NUM_CREATORS"
        );
```

So this check just repeats itself as there is no way for attacker to increase the number of creators somewhere in between creating an art piece and then creating a verbId.

### Recommendation

Remove the check from the `VerbsToken` contract.

## [N-04] Hard-coded gas limit can lead to tx revert in some cases 

There protocol several times uses hardcoded gas limit that depending on the complexity of the transaction or gas cost changes in Ethereum could result in failed transactions:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L429
```
success := call(50000, _to, _amount, 0, 0, 0, 0)
```

### Recommendation

Don't use the hard-coded gas limit.

## `AuctionHouse` is not forced to be `minter` inside of `VerbsToken` when initializing 

Inside of the protocol, only `AuctionHouse` can call `mint()` function but the address of the auction contract is not enforced in the `initialize()` function which may lead to unexpected mistakes:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L153
```
  minter = _minter;
```

The same thing applies to the `CultureIndex` and `VerbsToken` contracts and `dropper` role.

### Recommendation

Enforce the dropper and minter to be the `VerbsToken` and `CultureIndex` contracts.


## [N-05]  Inconsistent checks in `AuctionHouse` 

When calling `settleAuction()` in `AuctionHouse` smart contract, there are several if-statements and else-statements inside of which there is some inconsistency:

1. The first if-statement checks whether the `address(this).balance < reservePrice`. But this is not possible (if the aforementioned attack with forced sent ETH is not realized) as `msg.value` that the bidder provides in `createBid()` is always greater than or equal to `reservePrice`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L179
```
require(msg.value >= reservePrice, "Must send at least reservePrice");
```

2. The first else-statement is realized when `address(this).balance >= reservePrice`. But if the contract suggests that every ETH will be sent from it and nothing excessive will be sent to it, the scenario in the following check is also not possible:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L358
```
     if (_auction.bidder == address(0))
```

As if `address(this).balance` >= `reservePrice`, it means there is a bidder. And if there is a bidder, it cannot be address(0).

