# Quality Assurance Report

| ID     | Issues                                                                                                               |
|--------|----------------------------------------------------------------------------------------------------------------------|
| [N-01] | Require check in getArtPieceById() function should use < instead of <=                                               |
| [N-02] | Redundant check in _mintTo() function can be removed                                                                 |
| [N-03] | Incorrect comment for mapping heap                                                                                   |
| [N-04] | No need to initialize unsigned integer to 0                                                                          |
| [N-05] | Remove redundant check from TokenEmitterRewards.sol                                                                  |
| [L-01] | PUSH0 opcode is not supported on BASE AND Optimism chains                                                            |
| [L-02] | Prefer prioritizing the left hand side of the tree than right in binary tree                                         |
| [L-03] | createPiece() can be spammed on cheap fee chains like Base and Optimism                                              |
| [L-04] | Return value of extractMax() not checked in function dropTopVotedPiece() in CultureIndex                             |
| [L-05] | 1.75% of ETH provided to buyToken() can be refunded by users to themselves when buying the ERC20 tokens              |
| [L-06] | Missing field and check for audio metadata type length to be greater than 0                                          |
| [L-07] | ERC20Votes liquidity fragmentation and art theft can occur if contracts are deployed across three different chains   |
| [L-08] | Use >= instead of > when using minVoteWeight as criteria to start voting                                             |
| [L-09] | Code does not follow spec and breaks soft invariant in pause/unpause functionalities                                 |
| [R-01] | Expose burn() function with access control in NontransferableERC20Votes.sol for any future supply mechanism purposes |
| [R-02] | Consider removing the `_minCreatorRateBps > minCreatorRateBps` check                                                 |

## [N-01] Require check in getArtPieceById() function should use < instead of <=

Check should be < and not <= since mint happens following which an increment occurs for the next token which will be minted. 
```solidity
File: VerbsToken.sol
274:     function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
275:         require(verbId <= _currentVerbId, "Invalid piece ID"); 
276:         return artPieces[verbId];
277:     }
```
## [N-02] Redundant check in _mintTo() function can be removed

Check not required since when a piece is created, it undergoes this check.
```solidity
File: VerbsToken.sol
287:         require( 
288:             artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
289:             "Creator array must not be > MAX_NUM_CREATORS"
290:         );
```
## [N-03] Incorrect comment for mapping heap 

Correct struct to mapping in the notice tag on Line 64.
```solidity
File: MaxHeap.sol
64:     /// @notice Struct to represent an item in the heap by it's ID
66:     mapping(uint256 => uint256) public heap; 
```
## [N-04] No need to initialize unsigned integer to 0

no need to initialize to 0 since it is the default value of unsigned integers.
```solidity
File: MaxHeap.sol
67:     uint256 public size = 0;
```
## [N-05] Remove redundant check from TokenEmitterRewards.sol

The check below will always return false i.e. we never revert since computeTotalReward() always returns 2.5% of the msgValue. Now since msgValue can never be less than 2.5% of itself, the check can be removed.
```solidity
File: TokenEmitterRewards.sol
19:         if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();
```

## [L-01] PUSH0 opcode is not supported on BASE AND Optimism chains

All contracts in [scope.txt](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/scope.txt) use mostly v0.8.22

Make sure to either use version lower than 0.8.20 or set evm-version to Paris to deploy on the above mentioned chains.

## [L-02] Prefer prioritizing the left hand side of the tree than right in binary tree

In the below code snippet, we can observe that if the leftValue > rightValue, the left side is preferred as expected. But for the case where leftValue = rightValue, currently the right hand side of the tree is prioritized when swapping. This can lead to some edge case scenarios where the property of ACBT (almost complete binary tree) can be broken. Thus use >= to prioritize the left side of the tree.
```solidity
File: MaxHeap.sol
109:             if (leftValue > rightValue) { 
110:                 swap(pos, left);
111:                 maxHeapify(left);
112:             } else {
113:                 swap(pos, right);
114:                 maxHeapify(right);
115:             }
```
## [L-03] createPiece() can be spammed on cheap fee chains like Base and Optimism

Currently, there is not limit on how many pieces can be inserted into the max heap tree. This can lead to a DOS in the max heap tree down the line (due to unnecessary mountain of heapify operations required to be performed by legitimate new elements inserted) and frontend spamming if too many pieces are brought into existence. Additionally, since this codebase will be deployed on Base and Optimism, which have cheaper fees, the resources to perform such an attack increases.

To resolve this issue, consider adding a rate limiter per address, which would limit users from creating more pieces in a certain amount of time. For example, not more than 20-25 pieces per day.

## [L-04] Return value of extractMax() not checked in function dropTopVotedPiece() in CultureIndex

The extractMax() function returns two uint256s i.e. the itemId at the top and its value, but neither of them are checked in the dropTopVotedPiece() function.
```solidity
File: MaxHeap.sol
160:     function extractMax() external onlyAdmin returns (uint256, uint256) {
161:         require(size > 0, "Heap is empty"); 
162: 
163:         uint256 popped = heap[0]; 
164:         heap[0] = heap[--size]; 
165:         maxHeapify(0); 
166: 
167:         return (popped, valueMapping[popped]); 
168:     }
```

## [L-05]  1.75% of ETH provided to buyToken() can be refunded by users to themselves when buying the ERC20 tokens

Buyers of the ERC20 tokens can pass in their addresses as protocolRewardsRecipients. This would refund them their 1.75% of ETH through the ProtocolRewards contract.
```solidity
File: ERC20TokenEmitter.sol
152:     function buyToken(
153:         address[] calldata addresses,
154:         uint[] calldata basisPointSplits,
155:         ProtocolRewardAddresses calldata protocolRewardsRecipients 
156:     ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
```
## [L-06] Missing field and check for audio metadata type length to be greater than 0

Missing field in metadata for audio as "string audio" and thus missing check for it in the function below.
```solidity
File: CultureIndex.sol
159:     function validateMediaType(ArtPieceMetadata calldata metadata) internal pure {
160:         require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");
161: 
162:         if (metadata.mediaType == MediaType.IMAGE)
163:             require(bytes(metadata.image).length > 0, "Image URL must be provided");
164:         else if (metadata.mediaType == MediaType.ANIMATION)
165:             require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");
166:         else if (metadata.mediaType == MediaType.TEXT)
167:             require(bytes(metadata.text).length > 0, "Text must be provided");
168:          
169:     }
```

## [L-07] ERC20Votes liquidity fragmentation and art theft can occur if contracts are deployed across three different chains

The issue is that currently all ERC20Votes tokens are non transferrable, thus limiting their usage as wrapped assets for bridging across chains. Each of the contracts will have their independent states, which allows for art pieces to be copied and sold across other chains.

For art theft - consider adding artist signatures as a part of the ArtPieceMetadata.

## [L-08] Use >= instead of > when using minVoteWeight as criteria to start voting

Minimum values usually represent the value required to cast a vote. But currently only > is considered instead of >=. This breaks a soft invariant of the protocol indirectly. Additionally users having the minimum amount of voting weight will not be allowed to vote, which is a loss for the creators of the art piece and the overall community supporting that art piece.
```solidity
File: CultureIndex.sol
316: require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");
```

## [L-09] Code does not follow spec and breaks soft invariant in pause/unpause functionalities

The function pause() can be called even when the function is already paused. The same goes for unpause(). Thus, this does not follow the expected code spec.
```solidity
File: AuctionHouse.sol
203:     /**
204:      * @notice Pause the Verbs auction house.
205:      * @dev This function can only be called by the owner when the
206:      * contract is unpaused. While no new auctions can be started when paused,
207:      * anyone can settle an ongoing auction.
208:      */
209:  
210:     function pause() external override onlyOwner {
211:         _pause();
212:     }
```

## [R-01] Expose burn() function with access control in NontransferableERC20Votes.sol for any future supply mechanism purposes

Currently in the NontransferableERC20Votes contract, only the mint() function has been externally exposed, which is called from the ERC20TokenEmitter. Although not necessary, it would be good to expose the burn() function with the onlyOwner access control as well, which could be used as some mintpass mechanism or supply mechanism in the future.

## [R-02] Consider removing the `_minCreatorRateBps > minCreatorRateBps`  check

The below check should be removed in function setMinCreatorRateBps() since it limits the new minimum to always be greater. This is not a logical check since the the function itself is used to set the minCreatorRateBps.
```solidity
File: AuctionHouse.sol
241:         require(
242:             _minCreatorRateBps > minCreatorRateBps,
243:             "Min creator rate must be greater than previous minCreatorRateBps"
244:         );
```