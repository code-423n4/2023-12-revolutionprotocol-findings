**Note:** I've removed the specific instances from the bot race and 4naly3er

## Summary

### Medium Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[M&#x2011;01](#m01-_safemint-should-be-used-rather-than-_mint-wherever-possible)] | `_safeMint()` should be used rather than `_mint()` wherever possible | 1 | 

Total: 1 instances over 1 issues

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-payable-function-does-not-transfer-eth)] | `payable` function does not transfer Eth | 2 | 
| [[L&#x2011;02](#l02-code-does-not-follow-the-best-practice-of-check-effects-interaction)] | Code does not follow the best practice of check-effects-interaction | 34 | 
| [[L&#x2011;03](#l03-input-array-lengths-may-differ)] | Input array lengths may differ | 3 | 
| [[L&#x2011;04](#l04-upgradeable-contract-not-initialized)] | Upgradeable contract not initialized | 7 | 

Total: 46 instances over 4 issues

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-constants-should-be-defined-rather-than-using-magic-numbers)] | `constant`s should be defined rather than using magic numbers | 6 | 
| [[N&#x2011;02](#n02-public-functions-not-called-by-the-contract-should-be-declared-external-instead)] | `public` functions not called by the contract should be declared `external` instead | 10 | 
| [[N&#x2011;03](#n03-common-functions-should-be-refactored-to-a-common-base-contract)] | Common functions should be refactored to a common base contract | 1 | 
| [[N&#x2011;04](#n04-consider-making-contracts-upgradeable)] | Consider making contracts `Upgradeable` | 1 | 
| [[N&#x2011;05](#n05-consider-using-a-struct-rather-than-having-many-function-input-parameters)] | Consider using a `struct` rather than having many function input parameters | 2 | 
| [[N&#x2011;06](#n06-consider-using-descriptive-constants-when-passing-zero-as-a-function-argument)] | Consider using descriptive `constant`s when passing zero as a function argument | 9 | 
| [[N&#x2011;07](#n07-consider-using-named-function-arguments)] | Consider using named function arguments | 16 | 
| [[N&#x2011;08](#n08-consider-using-named-returns)] | Consider using named returns | 38 | 
| [[N&#x2011;09](#n09-custom-errors-should-be-used-rather-than-revertrequire)] | Custom errors should be used rather than `revert()`/`require()` | 4 | 
| [[N&#x2011;10](#n10-function-state-mutability-can-be-restricted-to-pure)] | Function state mutability can be restricted to `pure` | 2 | 
| [[N&#x2011;11](#n11-function-state-mutability-can-be-restricted-to-view)] | Function state mutability can be restricted to `view` | 2 | 
| [[N&#x2011;12](#n12-large-multiples-of-ten-should-use-scientific-notation)] | Large multiples of ten should use scientific notation | 2 | 
| [[N&#x2011;13](#n13-missing-event-for-critical-parameter-change)] | Missing event for critical parameter change | 3 | 
| [[N&#x2011;14](#n14-natspec-error-declarations-should-have-dev-tags)] | NatSpec: Error declarations should have `@dev` tags | 2 | 
| [[N&#x2011;15](#n15-natspec-error-declarations-should-have-notice-tags)] | NatSpec: Error declarations should have `@notice` tags | 2 | 
| [[N&#x2011;16](#n16-natspec-function-return-tag-is-missing)] | NatSpec: Function `@return` tag is missing | 12 | 
| [[N&#x2011;17](#n17-natspec-non-public-state-variable-declarations-should-use-dev-tags)] | NatSpec: Non-public state variable declarations should use `@dev` tags | 5 | 
| [[N&#x2011;18](#n18-not-using-the-named-return-variables-anywhere-in-the-function-is-confusing)] | Not using the named return variables anywhere in the function is confusing | 5 | 
| [[N&#x2011;19](#n19-not-using-the-named-return-variables-anywhere-in-the-function-is-confusing)] | Not using the named return variables anywhere in the function is confusing | 5 | 
| [[N&#x2011;20](#n20-style-guide-contract-does-not-follow-the-solidity-style-guides-suggested-layout-ordering)] | Style guide: Contract does not follow the Solidity style guide's suggested layout ordering | 1 | 
| [[N&#x2011;21](#n21-style-guide-control-structures-do-not-follow-the-solidity-style-guide)] | Style guide: Control structures do not follow the Solidity Style Guide | 27 | 
| [[N&#x2011;22](#n22-style-guide-extraneous-whitespace)] | Style guide: Extraneous whitespace | 59 | 
| [[N&#x2011;23](#n23-style-guide-non-externalpublic-variable-names-should-begin-with-an-underscore)] | Style guide: Non-`external`/`public` variable names should begin with an underscore | 11 | 
| [[N&#x2011;24](#n24-style-guide-top-level-declarations-should-be-separated-by-at-least-two-lines)] | Style guide: Top-level declarations should be separated by at least two lines | 1 | 
| [[N&#x2011;25](#n25-the-nonreentrant-modifier-should-occur-before-all-other-modifiers)] | The `nonReentrant` `modifier` should occur before all other modifiers | 1 | 
| [[N&#x2011;26](#n26-use-_disableinitializers-in-the-constructor-body-rather-than-using-the-initializer-modifier)] | Use `_disableInitializers()` in the constructor body, rather than using the `initializer` modifier | 6 | 
| [[N&#x2011;27](#n27-use-of-override-is-unnecessary)] | Use of `override` is unnecessary | 5 | 
| [[N&#x2011;28](#n28-variables-need-not-be-initialized-to-zero)] | Variables need not be initialized to zero | 7 | 

Total: 245 instances over 28 issues





## Medium Risk Issues


### [M&#x2011;01] `_safeMint()` should be used rather than `_mint()` wherever possible
`_mint()` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function. In the cases below, `_mint()` does not call `ERC721TokenReceiver.onERC721Received()` on the recipient.

*There is one instance of this issue:*

```solidity
File: src/VerbsToken.sol

310:             _mint(to, verbId);

```
*GitHub*: [310](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L310-L310)


## Low Risk Issues


### [L&#x2011;01] `payable` function does not transfer Eth
Funds sent along with the call to this function will be lost. Unless this is a gas-saving optimization, the `payable` keyword should be removed

*There are 2 instances of this issue:*

```solidity
File: src/ERC20TokenEmitter.sol

64       constructor(
65           address _manager,
66           address _protocolRewards,
67           address _protocolFeeRecipient
68:      ) payable TokenEmitterRewards(_protocolRewards, _protocolFeeRecipient) initializer {

```
*GitHub*: [64](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L64-L68)

```solidity
File: src/abstract/TokenEmitter/TokenEmitterRewards.sol

7        constructor(
8            address _protocolRewards,
9            address _revolutionRewardRecipient
10:      ) payable RewardSplits(_protocolRewards, _revolutionRewardRecipient) {}

```
*GitHub*: [7](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L7-L10)



### [L&#x2011;02] Code does not follow the best practice of check-effects-interaction
Code should follow the best-practice of [check-effects-interaction](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.

*There are 34 instances of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit insert() called prior to this assignment in createPiece(), via: createPiece()
225:         newPiece.pieceId = pieceId;

/// @audit insert() called prior to this assignment in createPiece(), via: createPiece()
226          newPiece.totalVotesSupply = _calculateVoteWeight(
227              erc20VotingToken.totalSupply(),
228              erc721VotingToken.totalSupply()
229:         );

/// @audit totalSupply() called prior to this assignment in createPiece(), via: createPiece()
/// @audit insert() called prior to this assignment in createPiece(), via: createPiece()
230:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();

/// @audit totalSupply() called prior to this assignment in createPiece(), via: createPiece()
/// @audit insert() called prior to this assignment in createPiece(), via: createPiece()
231:         newPiece.metadata = metadata;

/// @audit totalSupply() called prior to this assignment in createPiece(), via: createPiece()
/// @audit insert() called prior to this assignment in createPiece(), via: createPiece()
232:         newPiece.sponsor = msg.sender;

/// @audit totalSupply() called prior to this assignment in createPiece(), via: createPiece()
/// @audit insert() called prior to this assignment in createPiece(), via: createPiece()
233:         newPiece.creationBlock = block.number;

/// @audit totalSupply() called prior to this assignment in createPiece(), via: createPiece()
/// @audit insert() called prior to this assignment in createPiece(), via: createPiece()
234:         newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;

/// @audit getPastVotes() called prior to this assignment in _vote(), via: _vote(), _getPastVotes()
316:         votes[pieceId][voter] = Vote(voter, weight);

/// @audit getPastVotes() called prior to this assignment in _vote(), via: _vote(), _getPastVotes()
317:         totalVoteWeights[pieceId] += weight;

/// @audit getMax() called prior to this assignment in dropTopVotedPiece(), via: dropTopVotedPiece(), getTopVotedPiece(), topVotedPieceId()
/// @audit size() called prior to this assignment in dropTopVotedPiece(), via: dropTopVotedPiece(), getTopVotedPiece(), topVotedPieceId()
526:         pieces[piece.pieceId].isDropped = true;

```
*GitHub*: [225](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L225-L225), [226](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L226-L229), [230](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L230-L230), [230](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L230-L230), [231](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L231-L231), [231](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L231-L231), [232](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L232-L232), [232](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L232-L232), [233](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L233-L233), [233](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L233-L233), [234](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L234-L234), [234](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L234-L234), [316](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L316-L316), [317](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L317-L317), [526](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L526-L526), [526](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L526-L526)

```solidity
File: src/ERC20TokenEmitter.sol

/// @audit yToX() called prior to this assignment in buyToken(), via: buyToken(), getTokenQuoteForEther()
/// @audit depositRewards() called prior to this assignment in buyToken(), via: buyToken(), _handleRewardsAndGetValueToSend(), _depositPurchaseRewards()
187:         emittedTokenWad += totalTokensForBuyers;

/// @audit yToX() called prior to this assignment in buyToken(), via: buyToken(), getTokenQuoteForEther()
/// @audit depositRewards() called prior to this assignment in buyToken(), via: buyToken(), _handleRewardsAndGetValueToSend(), _depositPurchaseRewards()
188:         if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

```
*GitHub*: [187](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L187-L187), [187](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L187-L187), [188](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L188-L188), [188](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L188-L188)

```solidity
File: src/VerbsToken.sol

/// @audit MAX_NUM_CREATORS() called prior to this assignment in _mintTo(), via: _mintTo()
/// @audit getTopVotedPiece() called prior to this assignment in _mintTo(), via: _mintTo()
298:             newPiece.pieceId = artPiece.pieceId;

/// @audit MAX_NUM_CREATORS() called prior to this assignment in _mintTo(), via: _mintTo()
/// @audit getTopVotedPiece() called prior to this assignment in _mintTo(), via: _mintTo()
299:             newPiece.metadata = artPiece.metadata;

/// @audit MAX_NUM_CREATORS() called prior to this assignment in _mintTo(), via: _mintTo()
/// @audit getTopVotedPiece() called prior to this assignment in _mintTo(), via: _mintTo()
300:             newPiece.isDropped = artPiece.isDropped;

/// @audit MAX_NUM_CREATORS() called prior to this assignment in _mintTo(), via: _mintTo()
/// @audit getTopVotedPiece() called prior to this assignment in _mintTo(), via: _mintTo()
301:             newPiece.sponsor = artPiece.sponsor;

/// @audit MAX_NUM_CREATORS() called prior to this assignment in _mintTo(), via: _mintTo()
/// @audit getTopVotedPiece() called prior to this assignment in _mintTo(), via: _mintTo()
302:             newPiece.totalERC20Supply = artPiece.totalERC20Supply;

/// @audit MAX_NUM_CREATORS() called prior to this assignment in _mintTo(), via: _mintTo()
/// @audit getTopVotedPiece() called prior to this assignment in _mintTo(), via: _mintTo()
303:             newPiece.quorumVotes = artPiece.quorumVotes;

/// @audit MAX_NUM_CREATORS() called prior to this assignment in _mintTo(), via: _mintTo()
/// @audit getTopVotedPiece() called prior to this assignment in _mintTo(), via: _mintTo()
304:             newPiece.totalVotesSupply = artPiece.totalVotesSupply;

```
*GitHub*: [298](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L298-L298), [298](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L298-L298), [299](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L299-L299), [299](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L299-L299), [300](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L300-L300), [300](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L300-L300), [301](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L301-L301), [301](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L301-L301), [302](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L302-L302), [302](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L302-L302), [303](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L303-L303), [303](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L303-L303), [304](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L304-L304), [304](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L304-L304)



### [L&#x2011;03] Input array lengths may differ
If the caller makes a copy-paste error, the lengths may be mismatched and an operation believed to have been completed may not in fact have been completed (e.g. if the array being iterated over is shorter than the one being indexed into).

*There are 3 instances of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit creatorArray[]
237:             newPiece.creators.push(creatorArray[i]);

/// @audit creatorArray[]
/// @audit creatorArray[]
244:             emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);

```
*GitHub*: [237](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L237-L237), [244](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L244-L244), [244](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L244-L244)



### [L&#x2011;04] Upgradeable contract not initialized
Upgradeable contracts are initialized via an initializer function rather than by a constructor. Leaving such a contract uninitialized may lead to it being taken over by a malicious user. Even if the function is currently empty, or its current functionality is accomplished in another way, a future version of OpenZeppelin may introduce new functionality in the function and, without a new security review, things will unexpectedly be vulnerable when fresh contract instances are deployed with the new version.

*There are 7 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

/// @audit missing __Ownable2Step_init()
39:   contract AuctionHouse is

```
*GitHub*: [39](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L39)

```solidity
File: src/CultureIndex.sol

/// @audit missing __Ownable2Step_init()
20:   contract CultureIndex is

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L20)

```solidity
File: src/ERC20TokenEmitter.sol

/// @audit missing __Ownable2Step_init()
17:   contract ERC20TokenEmitter is

```
*GitHub*: [17](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L17)

```solidity
File: src/MaxHeap.sol

/// @audit missing __Ownable2Step_init()
14:   contract MaxHeap is VersionedContract, UUPS, Ownable2StepUpgradeable, ReentrancyGuardUpgradeable {

```
*GitHub*: [14](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L14)

```solidity
File: src/NontransferableERC20Votes.sol

/// @audit missing __ERC20Votes_init()
/// @audit missing __Ownable2Step_init()
29:   contract NontransferableERC20Votes is Initializable, ERC20VotesUpgradeable, Ownable2StepUpgradeable {

```
*GitHub*: [29](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L29), [29](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L29)

```solidity
File: src/VerbsToken.sol

/// @audit missing __Ownable2Step_init()
33:   contract VerbsToken is

```
*GitHub*: [33](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L33)


## Non-critical Issues


### [N&#x2011;01] `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*There are 6 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

/// @audit 50000
426          assembly {
427              // Transfer ETH to the recipient
428              // Limit the call to 50,000 gas
429              success := call(50000, _to, _amount, 0, 0, 0, 0)
430:         }

```
*GitHub*: [426](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L426-L430)

```solidity
File: src/CultureIndex.sol

/// @audit 1e18
285:         return erc20Balance + (erc721Balance * erc721VotingTokenWeight * 1e18);

```
*GitHub*: [285](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L285-L285)

```solidity
File: src/libs/VRGDAC.sol

/// @audit 1e18
35:          decayConstant = wadLn(1e18 - _priceDecayPercent);

/// @audit 1e18
74:                                          wadPow(1e18 - priceDecayPercent, wadDiv(soldDifference, perTimeUnit))

/// @audit 1e18
91:                      wadPow(1e18 - priceDecayPercent, timeSinceStart - unsafeWadDiv(sold, perTimeUnit)) -

/// @audit 1e18
92:                          wadPow(1e18 - priceDecayPercent, timeSinceStart)

```
*GitHub*: [35](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L35-L35), [74](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L74-L74), [91](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L91-L91), [92](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L92-L92)



### [N&#x2011;02] `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There are 10 instances of this issue:*

```solidity
File: src/NontransferableERC20Votes.sol

87:       function decimals() public view virtual override returns (uint8) {

94:       function transfer(address, uint256) public virtual override returns (bool) {

108:      function transferFrom(address, address, uint256) public virtual override returns (bool) {

115:      function approve(address, uint256) public virtual override returns (bool) {

```
*GitHub*: [87](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L87), [94](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L94), [108](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L108), [115](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L115)

```solidity
File: src/VerbsToken.sol

177:      function mint() public override onlyMinter nonReentrant returns (uint256) {

184:      function burn(uint256 verbId) public override onlyMinter nonReentrant {

193:      function tokenURI(uint256 tokenId) public view override returns (string memory) {

201:      function dataURI(uint256 tokenId) public view override returns (string memory) {

```
*GitHub*: [177](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L177), [184](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L184), [193](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L193), [201](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L201)

```solidity
File: src/libs/VRGDAC.sol

47:       function xToY(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {

54:       function yToX(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {

```
*GitHub*: [47](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L47), [54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L54)



### [N&#x2011;03] Common functions should be refactored to a common base contract
The functions below have the same implementation as is seen in other files. The functions should be refactored into functions of a common base contract

*There is one instance of this issue:*

```solidity
File: src/MaxHeap.sol

181      function _authorizeUpgrade(address _newImpl) internal view override onlyOwner {
182          // Ensure the new implementation is a registered upgrade
183          if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
184:     }

```
*GitHub*: [181](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L181-L184)



### [N&#x2011;04] Consider making contracts `Upgradeable`
This allows for bugs to be fixed in production, at the expense of _significantly_ increasing centralization.

*There is one instance of this issue:*

```solidity
File: src/libs/VRGDAC.sol

11   contract VRGDAC {
12       /*//////////////////////////////////////////////////////////////
13                               VRGDA PARAMETERS
14       //////////////////////////////////////////////////////////////*/
15:  

```
*GitHub*: [11](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L11-L15)



### [N&#x2011;05] Consider using a `struct` rather than having many function input parameters
Often times, a subset of the parameters would be more clear if they were passed as a struct

*There are 2 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

113      function initialize(
114          address _erc721Token,
115          address _erc20TokenEmitter,
116          address _initialOwner,
117          address _weth,
118          IRevolutionBuilder.AuctionParams calldata _auctionParams
119:     ) external initializer {

```
*GitHub*: [113](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L113-L119)

```solidity
File: src/VerbsToken.sol

130      function initialize(
131          address _minter,
132          address _initialOwner,
133          address _descriptor,
134          address _cultureIndex,
135          IRevolutionBuilder.ERC721TokenParams memory _erc721TokenParams
136:     ) external initializer {

```
*GitHub*: [130](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L130-L136)



### [N&#x2011;06] Consider using descriptive `constant`s when passing zero as a function argument
Passing zero as a function argument can sometimes result in a security issue (e.g. passing zero as the slippage parameter). Consider using a `constant` variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons.

*There are 9 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

319:                 amount: 0,

322:                 bidder: payable(0),

404:                             builder: address(0),

405:                             purchaseReferral: address(0),

```
*GitHub*: [319](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L319-L319), [322](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L322-L322), [404](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L404-L404), [405](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L405-L405)

```solidity
File: src/ERC20TokenEmitter.sol

191:         (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));

196:             (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));

```
*GitHub*: [191](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L191-L191), [196](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L196-L196)

```solidity
File: src/MaxHeap.sol

161:         maxHeapify(0);

```
*GitHub*: [161](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L161-L161)

```solidity
File: src/NontransferableERC20Votes.sol

129:             revert ERC20InvalidReceiver(address(0));

131:         _update(address(0), account, value);

```
*GitHub*: [129](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L129-L129), [131](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L131-L131)



### [N&#x2011;07] Consider using named function arguments
When calling functions in external contracts with multiple arguments, consider using [named](https://docs.soliditylang.org/en/latest/control-structures.html#function-calls-with-named-parameters) function parameters, rather than positional ones.

*There are 16 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

400:                     creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: creatorsShare - ethPaidToCreators }(

454:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [400](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L400-L400), [454](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L454-L454)

```solidity
File: src/CultureIndex.sol

221:         maxHeap.insert(pieceId, 0);

295:                 erc20VotingToken.getPastVotes(account, blockNumber),

296:                 erc721VotingToken.getPastVotes(account, blockNumber)

322:         maxHeap.updateValue(pieceId, totalWeight);

545:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [221](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L221-L221), [295](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L295-L295), [296](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L296-L296), [322](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L322-L322), [545](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L545-L545)

```solidity
File: src/ERC20TokenEmitter.sol

109:         token.mint(_to, _amount);

242              vrgdac.xToY({
243                  timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
244                  sold: emittedTokenWad,
245                  amount: int(amount)
246:             });

259              vrgdac.yToX({
260                  timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
261                  sold: emittedTokenWad,
262                  amount: int(etherAmount)
263:             });

276              vrgdac.yToX({
277                  timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
278                  sold: emittedTokenWad,
279                  amount: int(((paymentAmount - computeTotalReward(paymentAmount)) * (10_000 - creatorRateBps)) / 10_000)
280:             });

```
*GitHub*: [109](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L109-L109), [242](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L242-L246), [259](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L259-L263), [276](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L276-L280)

```solidity
File: src/MaxHeap.sol

183:         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [183](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L183-L183)

```solidity
File: src/VerbsToken.sol

194:         return descriptor.tokenURI(tokenId, artPieces[tokenId].metadata);

202:         return descriptor.dataURI(tokenId, artPieces[tokenId].metadata);

330:         require(manager.isRegisteredUpgrade(_getImplementation(), _newImpl), "Invalid upgrade");

```
*GitHub*: [194](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L194-L194), [202](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L202-L202), [330](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L330-L330)

```solidity
File: src/abstract/RewardSplits.sol

80:          protocolRewards.depositRewards{ value: totalReward }(

```
*GitHub*: [80](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L80-L80)



### [N&#x2011;08] Consider using named returns
Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is ideal for named returns.

*There are 38 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/CultureIndex.sol

179:     function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {

209      function createPiece(
210          ArtPieceMetadata calldata metadata,
211          CreatorBps[] calldata creatorArray
212:     ) public returns (uint256) {

256:     function hasVoted(uint256 pieceId, address voter) external view returns (bool) {

265:     function getVotes(address account) external view override returns (uint256) {

274:     function getPastVotes(address account, uint256 blockNumber) external view override returns (uint256) {

284:     function _calculateVoteWeight(uint256 erc20Balance, uint256 erc721Balance) internal view returns (uint256) {

288:     function _getVotes(address account) internal view returns (uint256) {

292:     function _getPastVotes(address account, uint256 blockNumber) internal view returns (uint256) {

451:     function getPieceById(uint256 pieceId) public view returns (ArtPiece memory) {

461:     function getVote(uint256 pieceId, address voter) public view returns (Vote memory) {

470:     function getTopVotedPiece() public view returns (ArtPiece memory) {

478:     function pieceCount() external view returns (uint256) {

486:     function topVotedPieceId() public view returns (uint256) {

509:     function quorumVotes() public view returns (uint256) {

519:     function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {

```
*GitHub*: [179](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L179-L179), [209](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L209-L212), [256](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L256-L256), [265](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L265-L265), [274](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L274-L274), [284](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L284-L284), [288](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L288-L288), [292](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L292-L292), [451](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L451-L451), [461](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L461-L461), [470](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L470-L470), [478](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L478-L478), [486](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L486-L486), [509](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L509-L509), [519](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L519-L519)

```solidity
File: src/ERC20TokenEmitter.sol

112:     function totalSupply() public view returns (uint) {

117:     function decimals() public view returns (uint8) {

122:     function balanceOf(address _owner) public view returns (uint) {

```
*GitHub*: [112](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L112-L112), [117](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L117-L117), [122](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L122-L122)

```solidity
File: src/MaxHeap.sol

78:      function parent(uint256 pos) private pure returns (uint256) {

156:     function extractMax() external onlyAdmin returns (uint256, uint256) {

169:     function getMax() public view returns (uint256, uint256) {

```
*GitHub*: [78](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L78-L78), [156](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L156-L156), [169](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L169-L169)

```solidity
File: src/NontransferableERC20Votes.sol

87:      function decimals() public view virtual override returns (uint8) {

94:      function transfer(address, uint256) public virtual override returns (bool) {

108:     function transferFrom(address, address, uint256) public virtual override returns (bool) {

115:     function approve(address, uint256) public virtual override returns (bool) {

```
*GitHub*: [87](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L87-L87), [94](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L94-L94), [108](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L108-L108), [115](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L115-L115)

```solidity
File: src/VerbsToken.sol

161:     function contractURI() public view returns (string memory) {

177:     function mint() public override onlyMinter nonReentrant returns (uint256) {

193:     function tokenURI(uint256 tokenId) public view override returns (string memory) {

201:     function dataURI(uint256 tokenId) public view override returns (string memory) {

273:     function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {

281:     function _mintTo(address to) internal returns (uint256) {

```
*GitHub*: [161](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L161-L161), [177](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L177-L177), [193](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L193-L193), [201](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L201-L201), [273](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L273-L273), [281](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L281-L281)

```solidity
File: src/libs/VRGDAC.sol

47:      function xToY(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {

54:      function yToX(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {

86:      function pIntegral(int256 timeSinceStart, int256 sold) internal view returns (int256) {

```
*GitHub*: [47](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L47-L47), [54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L54-L54), [86](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L86-L86)

```solidity
File: src/abstract/RewardSplits.sol

40:      function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {

54:      function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {

66       function _depositPurchaseRewards(
67           uint256 paymentAmountWei,
68           address builderReferral,
69           address purchaseReferral,
70           address deployer
71:      ) internal returns (uint256) {

```
*GitHub*: [40](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40-L40), [54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L54-L54), [66](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L66-L71)

```solidity
File: src/abstract/TokenEmitter/TokenEmitterRewards.sol

12       function _handleRewardsAndGetValueToSend(
13           uint256 msgValue,
14           address builderReferral,
15           address purchaseReferral,
16           address deployer
17:      ) internal returns (uint256) {

```
*GitHub*: [12](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12-L17)

</details>





### [N&#x2011;09] Custom errors should be used rather than `revert()`/`require()`
Custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in `try`-`catch` blocks, and are easier to re-use and maintain.

*There are 4 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

421:         if (address(this).balance < _amount) revert("Insufficient balance");

441:             if (!wethSuccess) revert("WETH transfer failed");

```
*GitHub*: [421](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L421-L421), [441](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L441-L441)

```solidity
File: src/VerbsToken.sol

317:             revert("dropTopVotedPiece failed");

```
*GitHub*: [317](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L317-L317)

```solidity
File: src/abstract/RewardSplits.sol

30:          if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L30-L30)



### [N&#x2011;10] Function state mutability can be restricted to `pure`


*There are 2 instances of this issue:*

```solidity
File: src/NontransferableERC20Votes.sol

101:     function _transfer(address from, address to, uint256 value) internal override {

141:     function _approve(address owner, address spender, uint256 value) internal override {

```
*GitHub*: [101](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L101-L101), [141](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L141-L141)



### [N&#x2011;11] Function state mutability can be restricted to `view`
Note that when overriding, mutability is allowed to be changed to be [more](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) strict than the parent function's mutability

*There are 2 instances of this issue:*

```solidity
File: src/NontransferableERC20Votes.sol

101:     function _transfer(address from, address to, uint256 value) internal override {

141:     function _approve(address owner, address spender, uint256 value) internal override {

```
*GitHub*: [101](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L101-L101), [141](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L141-L141)



### [N&#x2011;12] Large multiples of ten should use scientific notation
Large multiples of ten should use scientific notation (e.g. `1e6`) rather than decimal literals (e.g. `1000000`), for readability

*There are 2 instances of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit 6_000
48:      uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%

```
*GitHub*: [48](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L48-L48)

```solidity
File: src/abstract/RewardSplits.sol

/// @audit 50_000
24:      uint256 public constant maxPurchaseAmount = 50_000 ether;

```
*GitHub*: [24](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L24-L24)



### [N&#x2011;13] Missing event for critical parameter change
Events help non-contract tools to track changes

*There are 3 instances of this issue:*

```solidity
File: src/MaxHeap.sol

/// @audit valueMapping:  insert()
/// @audit heap:  insert()
119:     function insert(uint256 itemId, uint256 value) public onlyAdmin {

/// @audit valueMapping:  updateValue()
136:     function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {

```
*GitHub*: [119](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L119-L119), [119](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L119-L119), [136](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L136-L136)



### [N&#x2011;14] NatSpec: Error declarations should have `@dev` tags
`@dev` is used to explain extra details to developers

*There are 2 instances of this issue:*

```solidity
File: src/NontransferableERC20Votes.sol

30:      error TRANSFER_NOT_ALLOWED();

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L30-L30)

```solidity
File: src/abstract/RewardSplits.sol

15:      error INVALID_ETH_AMOUNT();

```
*GitHub*: [15](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L15-L15)



### [N&#x2011;15] NatSpec: Error declarations should have `@notice` tags
`@notice` is used to explain to end users what the error does, and the compiler interprets `///` or `/**` comments (but not `//` or `/*`) as this tag if one wasn't explicitly provided

*There are 2 instances of this issue:*

```solidity
File: src/NontransferableERC20Votes.sol

30:      error TRANSFER_NOT_ALLOWED();

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L30-L30)

```solidity
File: src/abstract/RewardSplits.sol

15:      error INVALID_ETH_AMOUNT();

```
*GitHub*: [15](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L15-L15)



### [N&#x2011;16] NatSpec: Function `@return` tag is missing


*There are 12 instances of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit Missing '@return  '
288:     function _getVotes(address account) internal view returns (uint256) {

/// @audit Missing '@return  '
292:     function _getPastVotes(address account, uint256 blockNumber) internal view returns (uint256) {

```
*GitHub*: [288](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L288-L288), [292](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L292-L292)

```solidity
File: src/ERC20TokenEmitter.sol

/// @audit Missing '@return  '
112:     function totalSupply() public view returns (uint) {

/// @audit Missing '@return  '
117:     function decimals() public view returns (uint8) {

/// @audit Missing '@return  '
122:     function balanceOf(address _owner) public view returns (uint) {

```
*GitHub*: [112](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L112-L112), [117](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L117-L117), [122](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L122-L122)

```solidity
File: src/libs/VRGDAC.sol

/// @audit Missing '@return  '
47:      function xToY(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {

/// @audit Missing '@return  '
54:      function yToX(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {

/// @audit Missing '@return  '
86:      function pIntegral(int256 timeSinceStart, int256 sold) internal view returns (int256) {

```
*GitHub*: [47](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L47-L47), [54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L54-L54), [86](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L86-L86)

```solidity
File: src/abstract/RewardSplits.sol

/// @audit Missing '@return  '
40:      function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {

/// @audit Missing '@return  '
54:      function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {

/// @audit Missing '@return  '
66       function _depositPurchaseRewards(
67           uint256 paymentAmountWei,
68           address builderReferral,
69           address purchaseReferral,
70           address deployer
71:      ) internal returns (uint256) {

```
*GitHub*: [40](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40-L40), [54](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L54-L54), [66](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L66-L71)

```solidity
File: src/abstract/TokenEmitter/TokenEmitterRewards.sol

/// @audit Missing '@return  '
12       function _handleRewardsAndGetValueToSend(
13           uint256 msgValue,
14           address builderReferral,
15           address purchaseReferral,
16           address deployer
17:      ) internal returns (uint256) {

```
*GitHub*: [12](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12-L17)



### [N&#x2011;17] NatSpec: Non-public state variable declarations should use `@dev` tags
i.e. `@dev` [tags](https://docs.soliditylang.org/en/latest/natspec-format.html#tags). Note that since they're non-public, `@notice` is not the right tag to use.

*There are 5 instances of this issue:*

```solidity
File: src/CultureIndex.sol

85:      IRevolutionBuilder private immutable manager;

```
*GitHub*: [85](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L85-L85)

```solidity
File: src/ERC20TokenEmitter.sol

55:      IRevolutionBuilder private immutable manager;

```
*GitHub*: [55](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L55-L55)

```solidity
File: src/MaxHeap.sol

23:      IRevolutionBuilder private immutable manager;

```
*GitHub*: [23](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L23-L23)

```solidity
File: src/NontransferableERC20Votes.sol

37:      IRevolutionBuilder private immutable manager;

```
*GitHub*: [37](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L37-L37)

```solidity
File: src/VerbsToken.sol

109:     IRevolutionBuilder private immutable manager;

```
*GitHub*: [109](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L109-L109)



### [N&#x2011;18] Not using the named return variables anywhere in the function is confusing
Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.

*There are 5 instances of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit success
419       function _verifyVoteSignature(
420           address from,
421           uint256[] calldata pieceIds,
422           uint256 deadline,
423           uint8 v,
424           bytes32 r,
425           bytes32 s
426:      ) internal returns (bool success) {

```
*GitHub*: [419](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L419-L426)

```solidity
File: src/ERC20TokenEmitter.sol

/// @audit tokensSoldWad
152       function buyToken(
153           address[] calldata addresses,
154           uint[] calldata basisPointSplits,
155           ProtocolRewardAddresses calldata protocolRewardsRecipients
156:      ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {

/// @audit spentY
237:      function buyTokenQuote(uint256 amount) public view returns (int spentY) {

/// @audit gainedX
254:      function getTokenQuoteForEther(uint256 etherAmount) public view returns (int gainedX) {

/// @audit gainedX
271:      function getTokenQuoteForPayment(uint256 paymentAmount) external view returns (int gainedX) {

```
*GitHub*: [152](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L152-L156), [237](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L237), [254](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L254), [271](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L271)



### [N&#x2011;19] Not using the named return variables anywhere in the function is confusing
Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.

*There are 5 instances of this issue:*

```solidity
File: src/CultureIndex.sol

/// @audit success
419      function _verifyVoteSignature(
420          address from,
421          uint256[] calldata pieceIds,
422          uint256 deadline,
423          uint8 v,
424          bytes32 r,
425          bytes32 s
426:     ) internal returns (bool success) {

```
*GitHub*: [419](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L419-L426)

```solidity
File: src/ERC20TokenEmitter.sol

/// @audit tokensSoldWad
152      function buyToken(
153          address[] calldata addresses,
154          uint[] calldata basisPointSplits,
155          ProtocolRewardAddresses calldata protocolRewardsRecipients
156:     ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {

/// @audit spentY
237:     function buyTokenQuote(uint256 amount) public view returns (int spentY) {

/// @audit gainedX
254:     function getTokenQuoteForEther(uint256 etherAmount) public view returns (int gainedX) {

/// @audit gainedX
271:     function getTokenQuoteForPayment(uint256 paymentAmount) external view returns (int gainedX) {

```
*GitHub*: [152](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L152-L156), [237](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L237-L237), [254](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L254-L254), [271](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L271-L271)



### [N&#x2011;20] Style guide: Contract does not follow the Solidity style guide's suggested layout ordering
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be 1) Type declarations, 2) State variables, 3) Events, 4) Modifiers, and 5) Functions, but the contract(s) below do not follow this ordering

*There is one instance of this issue:*

```solidity
File: src/MaxHeap.sol

/// @audit function constructor came earlier
41        modifier onlyAdmin() {
42            require(msg.sender == admin, "Sender is not the admin");
43            _;
44:       }

```
*GitHub*: [41](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L41-L44)



### [N&#x2011;21] Style guide: Control structures do not follow the Solidity Style Guide
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide

*There are 27 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

441:              if (!wethSuccess) revert("WETH transfer failed");

192:          if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;

195:          if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);

199:          if (extended) emit AuctionExtended(_auction.verbId, _auction.endTime);

421:          if (address(this).balance < _amount) revert("Insufficient balance");

441:              if (!wethSuccess) revert("WETH transfer failed");

454:          if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [441](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L441), [192](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L192), [195](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L195), [199](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L199), [421](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L421), [441](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L441), [454](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L454)

```solidity
File: src/CultureIndex.sol

162           if (metadata.mediaType == MediaType.IMAGE)
163:              require(bytes(metadata.image).length > 0, "Image URL must be provided");

164           else if (metadata.mediaType == MediaType.ANIMATION)
165:              require(bytes(metadata.animationUrl).length > 0, "Animation URL must be provided");

166           else if (metadata.mediaType == MediaType.TEXT)
167:              require(bytes(metadata.text).length > 0, "Text must be provided");

377:          if (!success) revert INVALID_SIGNATURE();

377:          if (!success) revert INVALID_SIGNATURE();

404:              if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();

438:          if (from == address(0)) revert ADDRESS_ZERO();

441:          if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

545:          if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [162](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L162-L163), [164](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L164-L165), [166](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L166-L167), [377](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L377), [377](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L377), [404](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L404), [438](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L438), [441](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L441), [545](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L545)

```solidity
File: src/ERC20TokenEmitter.sol

188:          if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

```
*GitHub*: [188](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L188)

```solidity
File: src/MaxHeap.sol

102:          if (pos >= (size / 2) && pos <= size) return;

150:          } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify

183:          if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);

```
*GitHub*: [102](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L102), [150](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L150), [183](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L183)

```solidity
File: src/abstract/RewardSplits.sol

41:           if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

30:           if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");

41:           if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

74:           if (builderReferral == address(0)) builderReferral = revolutionRewardRecipient;

76:           if (deployer == address(0)) deployer = revolutionRewardRecipient;

78:           if (purchaseReferral == address(0)) purchaseReferral = revolutionRewardRecipient;

```
*GitHub*: [41](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41), [30](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L30), [41](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41), [74](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L74), [76](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L76), [78](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L78)

```solidity
File: src/abstract/TokenEmitter/TokenEmitterRewards.sol

18:           if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();

```
*GitHub*: [18](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18)



### [N&#x2011;22] Style guide: Extraneous whitespace
See the [whitespace](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#whitespace-in-expressions) section of the Solidity Style Guide

*There are 59 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/AuctionHouse.sol

26:   import { PausableUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/PausableUpgradeable.sol";

27:   import { ReentrancyGuardUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";

28:   import { IAuctionHouse } from "./interfaces/IAuctionHouse.sol";

29:   import { IVerbsToken } from "./interfaces/IVerbsToken.sol";

30:   import { IWETH } from "./interfaces/IWETH.sol";

31:   import { IERC20TokenEmitter } from "./interfaces/IERC20TokenEmitter.sol";

32:   import { ICultureIndex } from "./interfaces/ICultureIndex.sol";

33:   import { IRevolutionBuilder } from "./interfaces/IRevolutionBuilder.sol";

34:   import { Ownable2StepUpgradeable } from "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

36:   import { UUPS } from "./libs/proxy/UUPS.sol";

37:   import { VersionedContract } from "./version/VersionedContract.sol";

400                       creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: creatorsShare - ethPaidToCreators }(
401:                          vrgdaReceivers,

435:              IWETH(WETH).deposit{ value: _amount }();

```
*GitHub*: [26](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L26), [27](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L27), [28](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L28), [29](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L29), [30](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L30), [31](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L31), [32](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L32), [33](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L33), [34](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L34), [36](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L36), [37](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L37), [400](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L400-L401), [435](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L435)

```solidity
File: src/CultureIndex.sol

4:    import { Ownable2StepUpgradeable } from "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

5:    import { ReentrancyGuardUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";

7:    import { UUPS } from "./libs/proxy/UUPS.sol";

8:    import { VersionedContract } from "./version/VersionedContract.sol";

10:   import { IRevolutionBuilder } from "./interfaces/IRevolutionBuilder.sol";

12:   import { ERC20VotesUpgradeable } from "./base/erc20/ERC20VotesUpgradeable.sol";

13:   import { MaxHeap } from "./MaxHeap.sol";

14:   import { ICultureIndex } from "./interfaces/ICultureIndex.sol";

16:   import { ERC721CheckpointableUpgradeable } from "./base/ERC721CheckpointableUpgradeable.sol";

17:   import { EIP712Upgradeable } from "@openzeppelin/contracts-upgradeable/utils/cryptography/EIP712Upgradeable.sol";

18:   import { Strings } from "@openzeppelin/contracts/utils/Strings.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L4), [5](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L5), [7](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L7), [8](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L8), [10](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L10), [12](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L12), [13](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L13), [14](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L14), [16](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L16), [17](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L17), [18](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L18)

```solidity
File: src/ERC20TokenEmitter.sol

4:    import { ReentrancyGuardUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";

5:    import { TokenEmitterRewards } from "@collectivexyz/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol";

6:    import { Ownable2StepUpgradeable } from "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

7:    import { PausableUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/PausableUpgradeable.sol";

9:    import { VRGDAC } from "./libs/VRGDAC.sol";

10:   import { toDaysWadUnsafe } from "./libs/SignedWadMath.sol";

11:   import { Strings } from "@openzeppelin/contracts/utils/Strings.sol";

12:   import { NontransferableERC20Votes } from "./NontransferableERC20Votes.sol";

13:   import { IERC20TokenEmitter } from "./interfaces/IERC20TokenEmitter.sol";

15:   import { IRevolutionBuilder } from "./interfaces/IRevolutionBuilder.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L4), [5](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L5), [6](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L6), [7](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L7), [9](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L9), [10](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L10), [11](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L11), [12](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L12), [13](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L13), [15](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L15)

```solidity
File: src/MaxHeap.sol

4:    import { Ownable2StepUpgradeable } from "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

5:    import { ReentrancyGuardUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";

6:    import { IRevolutionBuilder } from "./interfaces/IRevolutionBuilder.sol";

8:    import { UUPS } from "./libs/proxy/UUPS.sol";

9:    import { VersionedContract } from "./version/VersionedContract.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L4), [5](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L5), [6](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L6), [8](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L8), [9](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L9)

```solidity
File: src/NontransferableERC20Votes.sol

20:   import { Ownable2StepUpgradeable } from "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

21:   import { PausableUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/PausableUpgradeable.sol";

22:   import { Initializable } from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

24:   import { ERC20VotesUpgradeable } from "./base/erc20/ERC20VotesUpgradeable.sol";

25:   import { EIP712Upgradeable } from "@openzeppelin/contracts-upgradeable/utils/cryptography/EIP712Upgradeable.sol";

27:   import { IRevolutionBuilder } from "./interfaces/IRevolutionBuilder.sol";

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L20), [21](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L21), [22](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L22), [24](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L24), [25](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L25), [27](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L27)

```solidity
File: src/VerbsToken.sol

20:   import { Ownable2StepUpgradeable } from "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

21:   import { ReentrancyGuardUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";

22:   import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";

24:   import { UUPS } from "./libs/proxy/UUPS.sol";

25:   import { VersionedContract } from "./version/VersionedContract.sol";

27:   import { ERC721CheckpointableUpgradeable } from "./base/ERC721CheckpointableUpgradeable.sol";

28:   import { IDescriptorMinimal } from "./interfaces/IDescriptorMinimal.sol";

29:   import { ICultureIndex } from "./interfaces/ICultureIndex.sol";

30:   import { IVerbsToken } from "./interfaces/IVerbsToken.sol";

31:   import { IRevolutionBuilder } from "./interfaces/IRevolutionBuilder.sol";

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L20), [21](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L21), [22](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L22), [24](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L24), [25](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L25), [27](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L27), [28](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L28), [29](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L29), [30](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L30), [31](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L31)

```solidity
File: src/libs/VRGDAC.sol

4:    import { wadExp, wadLn, wadMul, wadDiv, unsafeWadDiv, wadPow } from "./SignedWadMath.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/libs/VRGDAC.sol#L4)

```solidity
File: src/abstract/RewardSplits.sol

4:    import { IRevolutionProtocolRewards } from "../interfaces/IRevolutionProtocolRewards.sol";

80            protocolRewards.depositRewards{ value: totalReward }(
81:               builderReferral,

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L4), [80](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L80-L81)

```solidity
File: src/abstract/TokenEmitter/TokenEmitterRewards.sol

4:    import { RewardSplits } from "../RewardSplits.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L4)

</details>





### [N&#x2011;23] Style guide: Non-`external`/`public` variable names should begin with an underscore
According to the Solidity Style Guide, non-`external`/`public` variable names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

*There are 11 instances of this issue:*

```solidity
File: src/CultureIndex.sol

85:      IRevolutionBuilder private immutable manager;

```
*GitHub*: [85](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L85-L85)

```solidity
File: src/ERC20TokenEmitter.sol

55:      IRevolutionBuilder private immutable manager;

```
*GitHub*: [55](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L55-L55)

```solidity
File: src/MaxHeap.sol

23:      IRevolutionBuilder private immutable manager;

```
*GitHub*: [23](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L23-L23)

```solidity
File: src/NontransferableERC20Votes.sol

37:      IRevolutionBuilder private immutable manager;

```
*GitHub*: [37](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L37-L37)

```solidity
File: src/VerbsToken.sol

109:     IRevolutionBuilder private immutable manager;

```
*GitHub*: [109](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L109-L109)

```solidity
File: src/abstract/RewardSplits.sol

18:      uint256 internal constant DEPLOYER_REWARD_BPS = 25;

19:      uint256 internal constant REVOLUTION_REWARD_BPS = 75;

20:      uint256 internal constant BUILDER_REWARD_BPS = 100;

21:      uint256 internal constant PURCHASE_REFERRAL_BPS = 50;

26:      address internal immutable revolutionRewardRecipient;

27:      IRevolutionProtocolRewards internal immutable protocolRewards;

```
*GitHub*: [18](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L18-L18), [19](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L19-L19), [20](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L20-L20), [21](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L21-L21), [26](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L26-L26), [27](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/protocol-rewards/src/abstract/RewardSplits.sol#L27-L27)



### [N&#x2011;24] Style guide: Top-level declarations should be separated by at least two lines
And functions within contracts should be separate by a [single](https://docs.soliditylang.org/en/latest/style-guide.html#blank-lines) line

*There is one instance of this issue:*

```solidity
File: src/NontransferableERC20Votes.sol

27    import { IRevolutionBuilder } from "./interfaces/IRevolutionBuilder.sol";
28    
29:   contract NontransferableERC20Votes is Initializable, ERC20VotesUpgradeable, Ownable2StepUpgradeable {

```
*GitHub*: [27](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L27-L29)



### [N&#x2011;25] The `nonReentrant` `modifier` should occur before all other modifiers
This is a best-practice to protect against reentrancy in other modifiers

*There is one instance of this issue:*

```solidity
File: src/VerbsToken.sol

232:      ) external override onlyOwner nonReentrant whenDescriptorNotLocked {

```
*GitHub*: [232](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L232)



### [N&#x2011;26] Use `_disableInitializers()` in the constructor body, rather than using the `initializer` modifier


*There are 6 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

95:      constructor(address _manager) payable initializer {

```
*GitHub*: [95](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L95-L95)

```solidity
File: src/CultureIndex.sol

92:      constructor(address _manager) payable initializer {

```
*GitHub*: [92](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/CultureIndex.sol#L92-L92)

```solidity
File: src/ERC20TokenEmitter.sol

64       constructor(
65           address _manager,
66           address _protocolRewards,
67           address _protocolFeeRecipient
68:      ) payable TokenEmitterRewards(_protocolRewards, _protocolFeeRecipient) initializer {

```
*GitHub*: [64](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L68-L68)

```solidity
File: src/MaxHeap.sol

30:      constructor(address _manager) payable initializer {

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L30-L30)

```solidity
File: src/NontransferableERC20Votes.sol

44:      constructor(address _manager) payable initializer {

```
*GitHub*: [44](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L44-L44)

```solidity
File: src/VerbsToken.sol

116:     constructor(address _manager) payable initializer {

```
*GitHub*: [116](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L116-L116)



### [N&#x2011;27] Use of `override` is unnecessary
Starting with Solidity version [0.8.8](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding), using the `override` keyword when the function solely overrides an interface function, and the function doesn't exist in multiple base contracts, is unnecessary.

*There are 5 instances of this issue:*

```solidity
File: src/NontransferableERC20Votes.sol

87:      function decimals() public view virtual override returns (uint8) {

94:      function transfer(address, uint256) public virtual override returns (bool) {

108:     function transferFrom(address, address, uint256) public virtual override returns (bool) {

115:     function approve(address, uint256) public virtual override returns (bool) {

```
*GitHub*: [87](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L87-L87), [94](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L94-L94), [108](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L108-L108), [115](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/NontransferableERC20Votes.sol#L115-L115)

```solidity
File: src/VerbsToken.sol

193:     function tokenURI(uint256 tokenId) public view override returns (string memory) {

```
*GitHub*: [193](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L193-L193)



### [N&#x2011;28] Variables need not be initialized to zero
The default value for variables is zero, so initializing them to zero is superfluous.

*There are 7 instances of this issue:*

```solidity
File: src/AuctionHouse.sol

346:         uint256 creatorTokensEmitted = 0;

380:                 uint256 ethPaidToCreators = 0;

384:                     for (uint256 i = 0; i < numCreators; i++) {

```
*GitHub*: [346](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L346-L346), [380](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L380-L380), [384](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/AuctionHouse.sol#L384-L384)

```solidity
File: src/ERC20TokenEmitter.sol

205:         uint256 bpsSum = 0;

209:         for (uint256 i = 0; i < addresses.length; i++) {

```
*GitHub*: [205](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L205-L205), [209](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/ERC20TokenEmitter.sol#L209-L209)

```solidity
File: src/MaxHeap.sol

67:      uint256 public size = 0;

```
*GitHub*: [67](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/MaxHeap.sol#L67-L67)

```solidity
File: src/VerbsToken.sol

306:             for (uint i = 0; i < artPiece.creators.length; i++) {

```
*GitHub*: [306](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/70ee1972ea6d047ef079fcd4b1e8700a89118405/packages/revolution/src/VerbsToken.sol#L306-L306)
