# Revolution Protocol QA Report

| ID            | Title                                                                                                                                  | Severity       |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| [L-01](#l-01) | `VerbsToken` contract: changing the `descriptor` will leave the previously minted tokens un-renderd                                    | Low            |
| [L-02](#l-02) | `VerbsToken` contract: changing the minter during an ongoing auction might block the `AuctionHouse` contract                           | Low            |
| [L-03](#l-03) | `CulturalIndex.hasVoted` function doesn't check if the `pieceId` exists                                                                | Low            |
| [L-04](#l-04) | Users can't vote in the same block the artpiec is created                                                                              | Low            |
| [L-05](#l-05) | `ERC20TokenEmitter.buyToken` function: `creatorsAddress` and `treasury` can receive tokens without buying it                           | Low            |
| [L-06](#l-06) | `ERC20TokenEmitter.buyToken` function: totalTokensForCreators is added to the `emittedTokenWad` without checking if it's really minted | Low            |
| [L-07](#l-07) | The protocol might be eventually not gaining auction profits as the `minCreatorRateBps` can be increased only                          | Low            |
| [L-08](#l-08) | `ERC20TokenEmitter` contrat: `block.timestamp` can be manipulated by miners                                                            | Low            |
| [I-01](#l-01) | `CulturalIndex.createPiece`: `creatorArray` could have duplicate addresses                                                             | Informational  |
| [I-02](#l-02) | Wrong check placement in `ERC20TokenEmitter.buyToken` function                                                                         | Informational  |
| [I-03](#l-03) | `creatorDirectPayment` is sent to the `creatorsAddress` without checking if the address is not address(0)                              | Informational  |
| [I-04](#l-04) | Incorrect documentation for `MaxHeap.insert` function                                                                                  | Informational  |
| [I-05](#l-05) | `AuctionHouse._settleAuction` function contains excessive calling for a storage variable                                               | Informational  |
| [I-06](#l-06) | Incorrect documentation for `CulturIndex.getVotes` function                                                                            | Informational  |
| [I-07](#l-07) | Incorrect documentation for `CultureIndex.getPastVotes` function                                                                       | Informational  |
| [I-08](#l-08) | Redundant check in `CultureIndex.topVotedPieceId` function                                                                             | Informational  |
| [I-09](#l-09) | Redundant check in `CultureIndex._verifyVoteSignature` function                                                                        | Informational  |
| [I-10](#l-10) | Incorrect comment line in `_verifyVoteSignature` function                                                                              | Informational  |
| [I-11](#l-11) | A check needs to be refactored in `CultureIndex._vote`                                                                                 | Informational  |
| [I-12](#l-12) | `VerbsToken._authorizeUpgrade` function has a commited documentation<                                                                  | Informational  |
| [I-13](#l-13) | `ERC20TokenEmitter.setCreatorsAddress` function has a redundant modifier                                                               | Informational  |
| [R-01](#r-01) | Add a function to enable users from disabling their signatures                                                                         | Recommendation |
| [R-02](#r-01) | `CultureIndex` contract: signatures become invalid if one of the signed art pieces is dropped                                          | Recommendation |

# Summary

| Severity           | Description                                               | Instances |
| ------------------ | --------------------------------------------------------- | --------- |
| Low severity       | Vulnerabilities with medium-low impact and low likelihood | 8         |
| Informational      | Suggestions on best practices and code readability        | 13        |
| Low Recommendation | Design recommendations                                    | 2         |

# Low Severity Findings

## [L-01] `VerbsToken` contract: changing the `descriptor` will leave the previously minted tokens un-renderd<a id="l-01" ></a>

## Details

- In `VerbsToken` contract: the `descriptor` is the contract that's responsible of returning tokens uri to be rendered, and this contract can be changed via `VerbsToken.setDescriptor` function.
- So if this contract is changed; this will leave the previously minted verbs tokens unable to be rendered as the returned uri from calling `descriptor.tokenURI` will be invalid.

## Proof of Concept

[VerbsToken.tokenURI function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L193-L195)

```javascript
    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        return descriptor.tokenURI(tokenId, artPieces[tokenId].metadata);
    }
```

[VerbsToken.setDescriptor function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L230C1-L236C6)

```javascript
    function setDescriptor(
        IDescriptorMinimal _descriptor
    ) external override onlyOwner nonReentrant whenDescriptorNotLocked {
        descriptor = _descriptor;

        emit DescriptorUpdated(_descriptor);
    }
```

## Recommendation

Either lock the descriptor once initialized or add a mapping to save each minted verbs with the relevant descriptor contract that will render it.

## [L-02] `VerbsToken` contract: changing the minter during an ongoing auction might block the `AuctionHouse` contract<a id="l-02" ></a>

## Details

- `AuctionHouse` contract supposed to have the `minter` role in the `VerbsToken` contract; where it can mint new verbs token when an auction is created, and burn verbs token if the auction fails.

- So if the owner of the `VerbsToken` changes the `minter` role during an ongoing auction in the `AuctionHouse`, this might result in blocking the `AuctionHouse` if the auction failed as the contract can't burn the minted verbs token for an auction.

## Proof of Concept

[AuctionHouse.\_settleAuction function/L348-L361](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L348C5-L361C86)

```javascript
    if (address(this).balance < reservePrice) {
            // If contract balance is less than reserve price, refund to the last bidder
            if (_auction.bidder != address(0)) {
                _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
            }

            // And then burn the Noun
            verbs.burn(_auction.verbId);
        } else {
            //If no one has bid, burn the Verb
            if (_auction.bidder == address(0))
                verbs.burn(_auction.verbId);
                //If someone has bid, transfer the Verb to the winning bidder
            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);
```

## Recommendation

Prevent updating the minter role in `VerbsToken` if there's an ongoing auction.

## [L-03] `CulturalIndex.hasVoted` function doesn't check if the `pieceId` exists<a id="l-03" ></a>

## Details

`CulturalIndex.hasVoted` function returns whether an account has voted for an art piece or not regardless of the this piece being existent or not.

## Proof of Concept

[CultureIndex.hasVoted function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L256C1-L258C6)

```javascript
    function hasVoted(uint256 pieceId, address voter) external view returns (bool) {
        return votes[pieceId][voter].voterAddress != address(0);
    }
```

## Recommendation

Update `CultureIndex.hasVoted` function to check if the id exists or not:

```diff
    function hasVoted(uint256 pieceId, address voter) external view returns (bool) {
+       require(pieceId < _currentPieceId, "Invalid piece ID");
        return votes[pieceId][voter].voterAddress != address(0);
    }
```

## [L-04] Users can't vote in the same block the art piec is created<a id="l-04" ></a>

## Details

- In `CulturalIndex` contract: when a user votes on an art piece, the weight of his votes is calculated based on the weight of the past votes of ERC20VotingToken and ERC721VotingToken (which are governance token and verbs NFTs), where `_getPastVotes` is invoked with the address of the user and the blockNumber of the art piece, and this will invoke the `getPastVotes` that will check the blocknumber and retrieve the votes weight at the end of that blocknumber:

  ```javascript
      function _getPastVotes(address account, uint256 blockNumber) internal view returns (uint256) {
          return
              _calculateVoteWeight(
                  erc20VotingToken.getPastVotes(account, blockNumber),
                  erc721VotingToken.getPastVotes(account, blockNumber)
              );
      }
  ```

- But if the block number of the art piece is similar to the current block number; the function will revert and the user can't vote:

  ```javascript
      function getPastVotes(address account, uint256 timepoint) public view virtual returns (uint256) {
          VotesStorage storage $ = _getVotesStorage();
          uint48 currentTimepoint = clock();// this will return Time.blockNumber() which is SafeCast.toUint48(block.number)
          if (timepoint >= currentTimepoint) {
              revert ERC5805FutureLookup(timepoint, currentTimepoint);
          }
          return $._delegateCheckpoints[account].upperLookupRecent(SafeCast.toUint48(timepoint));
      }
  ```

## Proof of Concept

[CultureIndex.\_getPastVotes function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L292C1-L298C6)

```javascript
    function _getPastVotes(address account, uint256 blockNumber) internal view returns (uint256) {
        return
            _calculateVoteWeight(
                erc20VotingToken.getPastVotes(account, blockNumber),
                erc721VotingToken.getPastVotes(account, blockNumber)
            );
    }
```

[VotesUpgradeable.getPastVotes function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/VotesUpgradeable.sol#L121C1-L128C6)

```javascript
    function getPastVotes(address account, uint256 timepoint) public view virtual returns (uint256) {
        VotesStorage storage $ = _getVotesStorage();
        uint48 currentTimepoint = clock();
        if (timepoint >= currentTimepoint) {
            revert ERC5805FutureLookup(timepoint, currentTimepoint);
        }
        return $._delegateCheckpoints[account].upperLookupRecent(SafeCast.toUint48(timepoint));
    }
```

## Recommendation

Enable voting to start on the same timeblock of the created artpiece:

```diff
    function getPastVotes(address account, uint256 timepoint) public view virtual returns (uint256) {
        VotesStorage storage $ = _getVotesStorage();
        uint48 currentTimepoint = clock();
-       if (timepoint >= currentTimepoint) {
+       if (timepoint > currentTimepoint) {
            revert ERC5805FutureLookup(timepoint, currentTimepoint);
        }
        return $._delegateCheckpoints[account].upperLookupRecent(SafeCast.toUint48(timepoint));
    }
```

## [L-05] `ERC20TokenEmitter.buyToken` function: `creatorsAddress` and `treasury` can receive tokens without buying it<a id="l-05" ></a>

## Details

- In `ERC20TokenEmitter.buyToken` function: a check is made to ensuer that the caller is not the `creatorsAddress` or `treasury` address as they are allowed to have voting ERC20 tokens by minting them part of the bought tokens amount but not via direct buy:

  ```javascript
  //prevent treasury from paying itself
  require(msg.sender != treasury &&
    msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");
  ```

- But these addresses can receive tokens if their addresses are set in the `addresses` argument of the `buyToken` function; as there's no check on this addresses array if it has `creatorsAddress` or `treasury` addresses when minting them tokens:

  ```javascript
      for (uint256 i = 0; i < addresses.length; i++) {
              if (totalTokensForBuyers > 0) {
                  // transfer tokens to address
                  _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
              }
              bpsSum += basisPointSplits[i];
          }
  ```

## Proof of Concept

[ERC20TokenEmitter.buyToken function/ L209-L215](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L209-L215)

```javascript
       for (uint256 i = 0; i < addresses.length; i++) {
            if (totalTokensForBuyers > 0) {
                // transfer tokens to address
                _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
            }
            bpsSum += basisPointSplits[i];
        }
```

## Recommendation

Either remove this check that prevents `creatorsAddress` and `treasury` from buying tokens, or check the `addresses` array to skip minting if it contains any of these addresses.

## [L-06] `ERC20TokenEmitter.buyToken` function: totalTokensForCreators is added to the `emittedTokenWad` without checking if it's really minted<a id="l-06" ></a>

## Details

- In `ERC20TokenEmitter.buyToken` function: the values of the `totalTokensForCreators` and `totalTokensForBuyers` represent the amounts of voting tokens **to be minted** for these addresses and these values are added to the total `emittedTokenWad` before minting.

- But if the `creatorsAddress` is `address(0)`; then this address will not be minted the `totalTokensForCreators` amount:

  ```javascript
  if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
    _mint(creatorsAddress, uint256(totalTokensForCreators));
  }
  ```

- This will result in an incorrect value of `emittedTokenWad` as it has counted the un-minted amount of totalTokensForCreators tokens, and this will adversly affect the price of the voting token (the price will increase if the `emittedTokenWad` is increased to a value ahead of the intended amount to be minted per day).

## Proof of Concept

[ERC20TokenEmitter.buyToken function/L186-L203](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L186C2-L203C10)

```javascript
//Transfer ETH to treasury and update emitted
emittedTokenWad += totalTokensForBuyers;
if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators; // @audit: the amount is added before checking if it's minted

//some code...

//Mint tokens for creators
if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
  _mint(creatorsAddress, uint256(totalTokensForCreators));
}
```

## Recommendation

Update `buyToken` function to account for the `totalTokensForCreators` amount after minting it
to the `creatorsAddress`:

```diff
//Transfer ETH to treasury and update emitted
emittedTokenWad += totalTokensForBuyers;
-if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

//some code...

//Mint tokens for creators
if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
  _mint(creatorsAddress, uint256(totalTokensForCreators));
+ emittedTokenWad += totalTokensForCreators;
}
```

## [L-07] The protocol might be eventually not gaining auction profits as the `minCreatorRateBps` can be increased only<a id="l-07" ></a>

## Details

- The `creatorRateBps` represents the split of the winning bid that is reserved for the creator of the Verb in basis points, and this value can be changed by the DAO via `AuctionHouse.setCreatorRateBps` function; where the new value is checked if it's > `minCreatorRateBps` (which represents the all time minimum split of the winning bid that is reserved for the creator of the Verb in basis points) before being assigned.

  ```javascript
  require(_creatorRateBps >=
    minCreatorRateBps, "Creator rate must be greater than or equal to minCreatorRateBps");
  ```

- And the value of the `minCreatorRateBps` can be updated by the DAO via calling `AuctionHouse.setMinCreatorRateBps` function; but it was noticed that the minimum value can be updated if it's greater than the previous one:

  ```javascript
  require(_minCreatorRateBps >
    minCreatorRateBps, "Min creator rate must be greater than previous minCreatorRateBps");
  ```

- This will result in `minCreatorRateBps` never being decreased, and the `creatorRateBps` will follow this trend as well, so this will result in the protocol gaining less and less of bidding proceeds with each update of the `creatorRateBps` (as it will be incremented only if the `minCreatorRateBps` is updated ==> and the higher the `creatorRateBps` the lower the gains of the protocol from auctions).

## Proof of Concept

[AuctionHouse.setMinCreatorRateBps function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L233C1-L246C6)

```javascript
    function setMinCreatorRateBps(uint256 _minCreatorRateBps) external onlyOwner {
        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
        require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");

        //ensure new min rate cannot be lower than previous min rate
        require(
            _minCreatorRateBps > minCreatorRateBps,
            "Min creator rate must be greater than previous minCreatorRateBps"
        );

        minCreatorRateBps = _minCreatorRateBps;

        emit MinCreatorRateBpsUpdated(_minCreatorRateBps);
    }
```

## Recommendation

Update `setMinCreatorRateBps` function to accept values less than the previously assigned value:

```diff
    function setMinCreatorRateBps(uint256 _minCreatorRateBps) external onlyOwner {
        require(_minCreatorRateBps <= creatorRateBps, "Min creator rate must be less than or equal to creator rate");
        require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");

-       //ensure new min rate cannot be lower than previous min rate
-       require(
-           _minCreatorRateBps > minCreatorRateBps,
-           "Min creator rate must be greater than previous minCreatorRateBps"
-       );

        minCreatorRateBps = _minCreatorRateBps;

        emit MinCreatorRateBpsUpdated(_minCreatorRateBps);
    }
```

## [L-08] `ERC20TokenEmitter` contrat: `block.timestamp` can be manipulated by miners <a id="l-08" ></a>

## Details

- `ERC20TokenEmitter` contract functions use `block.timestamp` to evaluate the price of the ERC20 voting tokens, where the price depends on the number of sold tokens per day, and if the number is ahead a limit set by the `VRGDAC` contract then the price increases and if it's behind the scheduled limit then the price decreases.

- `block.timestamp` is used to calculate the passed days (`timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime)`), and this can be manupulated by miners to manipulate the price of the voting token:

  - if the sold token amount is ahead of the schedule: then the token price will increase, but if a malicious miner decreases `block.timestamp` , then the price would return normal.
  - if the sold token amount is ahead of the schedule: then the token price will increase, and if a malicious miner increases `block.timestamp` , then the price would be decreased
  - if the sold token amount is behind the schedule: then the token price will decrease, but if a malicious miner decreases `block.timestamp` , then the price would return normal.
  - if the sold token amount is behind the schedule: then the token price will decrease, and if a malicious miner increases `block.timestamp` , then the price would be decreased more.

## Proof of Concept

[ERC20TokenEmitter.buyTokenQuote function/L243](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L243)

```javascript
timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
```

[ERC20TokenEmitter.getTokenQuoteForEther function/L260](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L260)

```javascript
timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
```

[ERC20TokenEmitter.getTokenQuoteForPayment function/L277](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L277)

```javascript
timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
```

## Recommendation

`block.number` can be used instead of `block.timestamp` as it can't be manipulated by miners.

# Informational Findings

## [I-01] `CulturalIndex.createPiece`: `creatorArray` could have duplicate addresses<a id="i-01" ></a>

## Details

`CulturalIndex.createPiece` function enables anyone from adding art pieces to be voted on and auctioned, and the proceeds of the auction is distributed between the deployer (the one who added the art piece) and the creators which their addresses are appeneded to the art piece as the `pieces[pieceId].creators`, but it was noticed that the `creatorArray` is only checked if any of the addresses is a zero address, but there's no check for duplicate addresses.

## Proof of Concept

[CultureIndex.validateCreatorsArray function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L179C3-L193C6)

```javascript
  function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
        uint256 creatorArrayLength = creatorArray.length;
        //Require that creatorArray is not more than MAX_NUM_CREATORS to prevent gas limit issues
        require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

        uint256 totalBps;
        for (uint i; i < creatorArrayLength; i++) {
            require(creatorArray[i].creator != address(0), "Invalid creator address");
            totalBps += creatorArray[i].bps;
        }

        require(totalBps == 10_000, "Total BPS must sum up to 10,000");

        return creatorArrayLength;
    }
```

## Recommendation

Ensure that the `creatorArray` has no duplicate addresses.

## [I-02] Wrong check placement in `ERC20TokenEmitter.buyToken` function<a id="i-02" ></a>

## Details

A chek made in `ERC20TokenEmitter.buyToken` function to ensure that the `totalTokensForBuyers > 0` is placed inside the for-loop while it should only present once before entering the loop.

## Proof of Concept

[ERC20TokenEmitter.buyToken function/ L209-L215](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L209-L215)

```javascript
       for (uint256 i = 0; i < addresses.length; i++) {
            if (totalTokensForBuyers > 0) {
                // transfer tokens to address
                _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
            }
            bpsSum += basisPointSplits[i];
        }
```

## Recommendation

```diff
+   if (totalTokensForBuyers > 0) {
       for (uint256 i = 0; i < addresses.length; i++) {
-           if (totalTokensForBuyers > 0) {
                // transfer tokens to address
                _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
-           }
            bpsSum += basisPointSplits[i];
        }
+    }
```

## [I-03] `creatorDirectPayment` is sent to the `creatorsAddress` without checking if the address is not address(0)<a id="i-03" ></a>

## Details

In `ERC20TokenEmitter.buyToken` function; the `creatorDirectPayment` is sent directly to the `creatorsAddress` without checking if that address `!= address(0)` which will result in losing these sent payments.

## Proof of Concept

[ERC20TokenEmitter.buyToken function/L194-L198](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L194C1-L198C10)

```javascript
        //Transfer ETH to creators
        if (creatorDirectPayment > 0) {
            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed.");
        }
```

## Recommendation

```diff
        //Transfer ETH to creators
-       if (creatorDirectPayment > 0) {
+       if (creatorDirectPayment > 0 && creatorsAddress != address(0)) {
            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed.");
        }
```

## [I-04] Incorrect documentation for `MaxHeap.insert` function<a id="i-04" ></a>

## Details

The documentation of the function indicates that the heap will revert if it's full, while there's no maximum value set for the size of the heap to compare against to check if it's full or not:

    ```javascript
    /// @dev The function will revert if the heap is full
    ```

## Proof of Concept

[MaxHeap.insert function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L115C2-L130C6)

```javascript
   /// @notice Insert an element into the heap
    /// @dev The function will revert if the heap is full
    /// @param itemId The item ID to insert
    /// @param value The value to insert
    function insert(uint256 itemId, uint256 value) public onlyAdmin {
        heap[size] = itemId;
        valueMapping[itemId] = value; // Update the value mapping
        positionMapping[itemId] = size; // Update the position mapping

        uint256 current = size;
        while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
            swap(current, parent(current));
            current = parent(current);
        }
        size++;
    }
```

## Recommendation

Update the documentation to match function intention.

## [I-05] `AuctionHouse._settleAuction` function contains excessive calling for a storage variable<a id="i-05" ></a>

## Details

In `AuctionHouse._settleAuction` function: the `creatorsShare * entropyRateBps` is called with each creator payment calculation in a for-loop while it should be calculated only once and used:

    ```javascript
    uint256 paymentAmount = (creatorsShare _ entropyRateBps _ creator.bps) / (10_000 \* 10_000);
    ```

## Proof of Concept

[AuctionHouse.\_settleAuction function / L390](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L390)

```javascript
uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
```

## Recommendation

```diff
        if (creatorsShare > 0 && entropyRateBps > 0) {
+           uint256 totalEth=creatorsShare * entropyRateBps;
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
                        vrgdaReceivers[i] = creator.creator;
                        vrgdaSplits[i] = creator.bps;

                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
-                       uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);

+                       uint256 paymentAmount = (totalEth * creator.bps) / (10_000 * 10_000);
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
                    }
                }
```

## [I-06] Incorrect documentation for `CulturIndex.getVotes` function<a id="i-06" ></a>

## Details

The documentation of function indicates that it will return the voting power of a voter at the current block, while it actually returns the value in the most recent checkpoint which doesn't necessarily be the current block

    ```javascript
    @notice Returns the voting power of a voter at the current block.
    ```

## Proof of Concept

[CultureIndex.getVotes function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L260C3-L267C6)

```javascript
  /**
     * @notice Returns the voting power of a voter at the current block.
     * @param account The address of the voter.
     * @return The voting power of the voter.
     */
    function getVotes(address account) external view override returns (uint256) {
        return _getVotes(account);
    }
```

## Recommendation

Update the documentation to match function intention.

## [I-07] Incorrect documentation for `CultureIndex.getPastVotes` function<a id="i-07" ></a>

## Details

The documentation of the function indicates that it will return the voting power of a voter at the current block, while it actually returns the voting power at a specific blockNumber:

    ```javascript
     * @notice Returns the voting power of a voter at the current block.
    ```

## Proof of Concept

[CultureIndex.getPastVotes function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L269C1-L276C6)

```javascript
    /**
     * @notice Returns the voting power of a voter at the current block.
     * @param account The address of the voter.
     * @return The voting power of the voter.
     */
    function getPastVotes(address account, uint256 blockNumber) external view override returns (uint256) {
        return _getPastVotes(account, blockNumber);
    }
```

## Recommendation

Update the documentation to match function intention.

## [I-08] Redundant check in `CultureIndex.topVotedPieceId` function<a id="i-08" ></a>

## Details

The function checks if `maxHeap.size() > 0` before calling `maxHeap.getMax()`, while the same check is done in the `maxHeap.getMax()` function.

## Proof of Concept

[CultureIndex.topVotedPieceId function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L486C1-L491C6)

```javascript
    function topVotedPieceId() public view returns (uint256) {
        require(maxHeap.size() > 0, "Culture index is empty");
        //slither-disable-next-line unused-return
        (uint256 pieceId, ) = maxHeap.getMax();
        return pieceId;
    }
```

[MaxHeap.getMax function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L169C1-L172C6)

```javascript
    function getMax() public view returns (uint256, uint256) {
        require(size > 0, "Heap is empty");
        return (heap[0], valueMapping[heap[0]]);
    }
```

## Recommendation

Remove the redundant check in the `topVotedPieceId` function:

```diff
    function topVotedPieceId() public view returns (uint256) {
-       require(maxHeap.size() > 0, "Culture index is empty");
        //slither-disable-next-line unused-return
        (uint256 pieceId, ) = maxHeap.getMax();
        return pieceId;
    }
```

## [I-09] Redundant check in `CultureIndex._verifyVoteSignature` function<a id="i-09" ></a>

## Details

Thhere's a check in this function to ensure that `from` address is not zero address:

    ```javascript
    if (from == address(0)) revert ADDRESS_ZERO();
    ```

while this check is implicitly done again in the next line:

    ```javascript
    if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();
    ```

## Proof of Concept

[CultureIndex.\_verifyVoteSignature function/L438](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L438)

```javascript
if (from == address(0)) revert ADDRESS_ZERO();
```

## Recommendation

Remove this redundant check:

```diff
-if (from == address(0)) revert ADDRESS_ZERO();
```

## [I-10] Incorrect comment line in `_verifyVoteSignature` function<a id="i-10" ></a>

## Details

A comment indicating that the check is made on `to` address instead of `from`:

    ```javascript
    // Ensure to address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();
    ```

## Proof of Concept

[CultureIndex.\_verifyVoteSignature function/L437](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L437)

```javascript
// Ensure to address is not 0
```

## Recommendation

```diff
-   // Ensure to address is not 0
+   // Ensure from address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();
```

## [I-11] A check needs to be refactored in `CultureIndex._vote`<a id="i-11" ></a>

## Details

This line of code:

    ```javascript
    require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
    ```

can be refactored as follows to decrease the number of operations by one:

    ```javascript
    require(votes[pieceId][voter].voterAddress == address(0), "Already voted");
    ```

## Proof of Concept

[CultureIndex.\_vote function/L311](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L311)

```javascript
require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
```

## Recommendation

```diff
-require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
+require(votes[pieceId][voter].voterAddress == address(0), "Already voted");
```

## [I-12] `VerbsToken._authorizeUpgrade` function has a commited documentation<a id="i-12" ></a>

## Details

It seems like the function documentation is re-commited by mistake (as it matches the same documentation for that function all over the protocol contracts).

## Proof of Concept

[CultureIndex.\_authorizeUpgrade function documentation](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L325-L328)

```javascript
    // /// @notice Ensures the caller is authorized to upgrade the contract and that the new implementation is valid
    // /// @dev This function is called in `upgradeTo` & `upgradeToAndCall`
    // /// @param _newImpl The new implementation address
    function _authorizeUpgrade(address _newImpl) internal view override onlyOwner {
```

## Recommendation

Consider either removing the committed function documentation or removing the second commit.

## [I-13] `ERC20TokenEmitter.setCreatorsAddress` function has a redundant modifier<a id="i-13" ></a>

## Details

`ERC20TokenEmitter.setCreatorsAddress` function has `nonReentrant` modifier; while using this modifier is redundant in this function as it's a setter and doesn't make any extrenal calls.

## Proof of Concept

[ERC20TokenEmitter.setCreatorsAddress function](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L309)

```javascript
function setCreatorsAddress(address _creatorsAddress) external override onlyOwner nonReentrant {
```

## Recommendation

Consider removing `nonReentrant` modifier from `ERC20TokenEmitter.setCreatorsAddress` function.

# Recommendation Findings

## [R-01] Add a function to enable users from disabling their signatures<a id="r-01" ></a>

## Details

- `CulturalIndex` contract has a functionality that enables users to vote onbehalf of other users on specific art pieces by signatures; but if it was noticed that there's no functionality to revoke/dinvalidate signatures.

- Signatures got invalidated after using them (as the nonces of the user is increased and the votehash will result in a wrong recovered address) or after the deadline of that signature is passed.

- So whenever a user changes his mind on voting on a specific artpiece after creating a signature; he can't do so as there's no mechanism to invalidate signatures.

## Recommendation

Add a function in the `CulturalIndex` contract to invalidate signatures by increasing the nonce of the user:

```javascript
function invalidateSignature() public{
    nonces[msg.sender]++;
}
```

## [R-02] `CultureIndex` contract: signatures become invalid if one of the signed art pieces is dropped<a id="r-02" ></a>

## Details

- `CultureIndex` contract has a functionality to enable anyone from executing a vote on a specific art pieces by signature, and each signature has a deadline after which the signature becomes invalid.

- And in the `_vote` function; the art piece is checked if dropped (auctioned) before voting and if so; the transaction reverts:

  ```javascript
  require(!pieces[pieceId].isDropped, "Piece has already been dropped");
  ```

- So if the signature has any artpiece that has been dropped after the signature created; it becomes invalid before its deadline.

## Proof of Concept

[CultureIndex.\_vote function/L310](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L310)

```javascript
require(!pieces[pieceId].isDropped, "Piece has already been dropped");
```

## Recommendation

Update `_vote` function to return false instead of reverting on a dropped art piece.
