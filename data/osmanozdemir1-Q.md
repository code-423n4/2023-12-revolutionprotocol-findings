### Summary

* \[L-01\] Missing duplicate address checks in the `CultureIndex::validateCreatorsArray()` function can lead untrusted creators to steal from their collaborators
    
* \[L-02\] `CultureIndex::validateMediaType` doesn't validate the "audio" media type
    
* \[L-03\] A change in `erc721VotingTokenWeight` may create unfair situations for voting
    
* \[L-04\] The `AuctionHouse` contract should implement `IERC721Receiver` interface
    
* \[N-01\] Developer comments in the `NontransferableERC20Votes` contract are misleading
    
* \[N-02\] `TokenEmitterRewards::_handleRewardsAndGetValueToSend` doesn't have any form of NatSpec explanation
    
* \[N-03\] It would be better to document what happens if the top two art pieces have the same vote count in the `MaxHeap`
    

### \[L-01\] Missing duplicate address checks in the `CultureIndex::validateCreatorsArray()` function can lead untrusted creators to steal from their collaborators

[https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L179C2-L193C6](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L179C2-L193C6)

User-inputted creator arrays are validated via `validateCreatorsArray` function during the art piece creation. This function validates:

* Creators array doesn't have a zero address
    
* `totalBps` is 10\_000
    

However, this function lacks a duplicate address check. The same addresses can be provided multiple times in that array.

This issue will probably not cause an issue if there are only a few creators.

But the maximum creator count is 100 in this protocol. In the case of a huge art collaboration with tens of creators, it is much easier to sneak the same address twice into the array.

Let's assume there is an art collaboration with more than 60 creators. One creator, who calls the `createPiece()` function, is malicious and others are not aware of it. This creator can put his address twice in the array and still match the exact 10000 bps by clipping minimal bps amounts from other creators.

**Recommendation**: Consider adding a check for duplicate address in the `validateCreatorsArray` function.

---

### \[L-02\] `CultureIndex::validateMediaType` doesn't validate the "audio" media type

[https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L159C5-L168C6](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L159C5-L168C6)

`MediaType` enum includes "image", "animation", "audio", "text" and "other"

`CultureIndex::validateMediaType` function only validates whether "image", "animation" or "text" media data are not empty but it never validates the "audio" type.

```solidity
     /* Requirements:
     * - The media type must be one of the defined types in the MediaType enum.
-->  * - The corresponding media data must not be empty.
     */
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

**Recommendation:**

```diff
struct ArtPieceMetadata {
        string name;
        string description;
        MediaType mediaType;
        string image;
        string text;
        string animationUrl;
+       string audioUrl;
    }
--------
+        else if (metadata.mediaType == MediaType.AUDIO)
+            require(bytes(metadata.audioUrl).length > 0, "Audio URL must be provided");
```

---

### \[L-03\] A change in `erc721VotingTokenWeight` may create unfair situations for voting

[https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L292C1-L298C6](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L292C1-L298C6)

Users vote on art pieces based on their voting powers when the art piece is created. Users' voting power in that block is fetched with `_getPastVotes` function.

This protocol has both ERC20 and ERC721 voting tokens and ERC721 voting token has a [weight](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L45) compared to ERC20. Users' voting power is calculated based on their token balances and this ERC721 weight.

```solidity
//@audit this function checks votes in the past but according to today's current ERC721 weight. It should use the ERC721 weight in that past block's time
    function _getPastVotes(address account, uint256 blockNumber) internal view returns (uint256) {
        return
            _calculateVoteWeight(
                erc20VotingToken.getPastVotes(account, blockNumber),
                erc721VotingToken.getPastVotes(account, blockNumber)
            );
    }

----------------
    function _calculateVoteWeight(uint256 erc20Balance, uint256 erc721Balance) internal view returns (uint256) {
        return erc20Balance + (erc721Balance * erc721VotingTokenWeight * 1e18);
    }
```

As we can see above, user votes are calculated with the current `erc721VotingTokenWeight`. Even the votes in the past are also calculated according to today's `erc721VotingTokenWeight`.

This may create unfair situations if the `erc721VotingTokenWeight` is changed by the owner in the future since users can only vote once for the same art piece.

* Let's assume Alice and Bob have the same amount of erc20 and erc721 tokens at the time of `block X` and an art piece is created in that block.
    
* Alice voted for the art piece with her voting power based on `erc721VotingTokenWeight`.
    
* `erc721VotingTokenWeight` is updated.
    
* Bob voted for the art piece.
    

Normally they both should have the same voting power in the exact same block for the same art piece. However, the vote weights are not calculated according to the `erc721VotingTokenWeight` at the art piece creation block, they are calculated according to the current `erc721VotingTokenWeight`.

**Recommendation:** The protocol should track `erc721VotingTokenWeight` like checkpoints and use the `erc721VotingTokenWeight` at that block when fetching past votes in a specific block.

**Note**: This is only valid if the protocol plans to update `erc721VotingTokenWeight` in the future depending on the ERC20 vote token emission rate. The scenario above is not valid if this parameter is never going to be changed, but the parameter should be immutable if that's the case.

---

### \[L-04\] The `AuctionHouse` contract should implement `IERC721Receiver` interface

[https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L39C1-L46C2](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L39C1-L46C2)

```solidity
contract AuctionHouse is
    IAuctionHouse,
    VersionedContract,
    UUPS,
    PausableUpgradeable,
    ReentrancyGuardUpgradeable,
    Ownable2StepUpgradeable
{
```

Every day a `verbsToken` is minted and transferred to the `AuctionHouse` contract. The art piece NFT is transferred to the buyer after the auction is concluded.

In the current implementation, `verbsToken`s are minted with the regular [`_mint`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L310) method in the [`VerbsToken::_mintTo()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L281) function, not with the `_safeMint` method. That's why not implementing `IERC721Receiver` is not a problem in this case.

```solidity
file: VerbsToken.sol
       function _mintTo(address to) internal returns(uint256) {
            //...
310.-->           _mint(to, verbId);
            // ...
       }
```

However, it is better to use `_safeMint` when minting NFTs to `AuctionHouse`, and in that case the AuctionHouse contract must implement `IERC721Receiver` interface.

---

### \[N-01\] Developer comments in the `NontransferableERC20Votes` contract are misleading

[https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/NontransferableERC20Votes.sol#L16C65-L16C94](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/NontransferableERC20Votes.sol#L16C65-L16C94)

```solidity
     /*... 
-->    * By default, token balance does not account for voting power. This makes transfers cheaper. The downside is that it
       * requires users to delegate to themselves in order to activate checkpoints and have their voting power tracked.
     */
```

Developer comments on the `NontransferableERC20Votes` contract is directly copied from OpenZeppelin [ERC20VotesUpgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/6dfb5c683528ed5de06a125e9215d10de8909b3b/contracts/token/ERC20/extensions/ERC20VotesUpgradeable.sol#L21C1-L23C4).

However, tokens in this contract are non-transferable. So the comment *"...This makes transfers cheaper..."* is not correct for this contract and should be removed.

---

### \[N-02\] `TokenEmitterRewards::_handleRewardsAndGetValueToSend` doesn't have any form of NatSpec explanation

[https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12C5-L21C6](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12C5-L21C6)

```solidity
    function _handleRewardsAndGetValueToSend(
        uint256 msgValue,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {
        if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();

        return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
    }
```

Consider adding NatSpec comments for this function.

---

### \[N-03\] It would be better to document what happens if the top two art pieces have the same vote count in the `MaxHeap`

`MaxHeap` contract keeps track of the vote counts of the art pieces in the form of a binary tree and is updated whenever an art piece is voted. The highest-voted art piece is dropped from the `maxHeap` contract and auctioned every day.

Currently, there is no documentation or explanation of what happens if there is an edge-case situation where the top two pieces have the same vote counts.

The behavior of the `maxHeap` contract in the current implementation is to keep the current top-voted art piece in its place in case the second one reaches the same vote count. The latter art piece must pass the current top-voted art piece to get the first place and having the same vote count is not enough.

The protocol should explain this behavior to its users beforehand and document it.

**Note**: I submitted it this way with the assumption that this is the intended behavior.  
If the intention was to switch the top-voted piece when an art piece comes behind and reaches the same vote count, this means that the `maxHeapify` function doesn't work as intended.