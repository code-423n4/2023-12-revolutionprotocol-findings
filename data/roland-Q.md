# QA Report

| *Issue* | *Description*                                                                  |
|---------|--------------------------------------------------------------------------------|
| [L-01]  | No option to withdraw vote                                                     |
| [L-02]  | Quorum for existing piece cannot be changed                                    |
| [L-03]  | Token inflation gives advantage to new pieces                                  |
| [L-04]  | Vote allocation for ERC721 minted for active auction is counted in totalVotesSupply                                                                           |
| [L-05]  | Front-running createPiece                                                      |
| [L-06]  | block.number accuracy in an L2 deployment                                      |
| [N-01]  | Unnecessary double negation in require statement                               |
| [N-02]  | Code comment mismatch in getVote()                                             |
| [N-03]  | No ownership checks on metadata content                                        |
| [N-04]  | Code and error message mismatch on AuctionHouse initialization                 |

## [L-01] No option to withdraw vote

There is currently no option for a user to withdraw their vote in [CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol). Although this seems to be by design, in a voting process where art pieces are competing with each other and the available option pool can change dramatically every day, it would be reasonable to assume a user has the option to reevaluate their choice after a change in circumstances.

### **Recommended Mitigation Steps**

Adding a function to nullify votes but have them count towards quorum would fix this issue.

## [L-02] Quorum for existing piece cannot be changed

There is currently no option to change the quorum value for an art piece that has already been created in [CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol). This can lead to situations in which a more popular older piece with stricter quorum requirements stays in the contract, while a newer piece with fewer votes but more relaxed quorum requirements proceeds to auction.

### **Recommended Mitigation Steps**

Unify quorum across all art pieces. Instead of storing quorumVotes in the ArtPiece struct, calculate the value in ```dropTopVotedPiece()``` with ```piece.totalVotesSupply``` and current ```quorumVotesBPS```

```
ICultureIndex.ArtPiece memory piece = getTopVotedPiece();
- require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");
+ require(totalVoteWeights[piece.pieceId] >= (quorumVotesBPS * piece.totalVotesSupply) / 10_000, "Does not meet quorum votes to be dropped.");
```

## [L-03] Token inflation gives advantage to new pieces

With constant token inflation and the vote winner being determined by absolute votes, the system will give preference to newer pieces, as they could end up having orders of magnitude greater eligible voting numbers.

### **Recommended Mitigation Steps**

Reconsider snapshotting user voting power, allowing each piece to have the same potential number of votes. The governance attack risk is greatly minimised by the non-transferable nature of the ERC20 governance tokens, as well as the illiquidity of the ERC721 tokens.

## [L-04] Vote allocation for ERC721 minted for active auction is counted in totalVotesSupply

In [CultureIndex.sol#L228](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L228), totalVotesSupply includes the corresponding votes of the ERC721 currently being auctioned off, although it will not be able to used to vote for the created piece, even after the auction end. The impact on voting process and quorum can vary depending on ERC721 voting weight.

## [L-05] Front-running createPiece  

In [CultureIndex.sol#L228](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L209), a malicious user might monitor art piece submissions in ```createPiece()```, and front-run the transaction with the same metadata but different creator addresses, to give legitimacy to the front-ran piece and gain advantage in the vote or auction.

## [L-06] block.number accuracy in an L2 deployment

In an L2 deployment of the protocol, ```getPastVotes()``` might revert if voting on a newly created piece within the same L1 block as the art piece creation block in [CultureIndex.sol#L233](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L233)

## [N-01] Unnecessary double negation in require statement

Double negation in [CultureIndex.sol#L457](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L311) makes the statement unnecessarily confusing.

### **Recommended Mitigation Steps**

Improve readability by removing double negation.

```
- require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
+ require(votes[pieceId][voter].voterAddress == address(0), "Already voted");
```

## [N-02] Code comment mismatch in getVote()

While getVote() in [CultureIndex.sol#L457](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L457) returns a single vote struct, given a pieceId and voter address, the function comments describe the return value to be an array of Vote structs given a pieceId. Tests indicate the code to be correct and the comments to be wrong.

## [N-03] No ownership checks on metadata content

While anyone can permissionlessly submit art piece in [CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol), they can also submit duplicates of art already submitted or auctioned off, for the purpose of diluting or stealing votes.

## [N-04] Code and error message mismatch on AuctionHouse initialization

[AuctionHouse.sol#L131](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L131)

```
@ audit error message should be changed to "Creator rate must be greater than or equal to the minimum creator rate"
require(
  _auctionParams.creatorRateBps >= _auctionParams.minCreatorRateBps,
  "Creator rate must be greater than or equal to the creator rate"
);
```
