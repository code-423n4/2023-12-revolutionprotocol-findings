# Advanced Analysis Report for [Revolution](https://github.com/code-423n4/2023-12-revolutionprotocol)


## Overview
#### The Revolution is a NFT art protocol similar to [Nouns](https://nouns.wtf/) but with a difference that Revolution enhances accessibility to the governance tokens by allowing anyone to upload their art to the protocol. Every day, an auction is created for the top-voted art piece and the winner of the auction will earn a governance NFT (the art piece).
### the governance tokens: 
* ERC721 minted exclusively for the auction winner.
* ERC20  which can be purchased through the **buyToken** function. Additionally, ERC20 tokens are distributed to the creators of the art after the conclusion of each auction.

## Roles Analysis

here we will discuess each role in the system and what he can do and the incentive that derives him. 

1. **artPiece sponsor** : the address that will add the art piece into the protocol and pay the fees , in return he will recieve rewards (erc20 tokens).
2. **erc20TokenHolders, erc721TokenHolders** : the addresses that can vote(if they have enough weight) on artPiece to make it place in the topVoted place.
3. **artPiece creators** : the creators that contrubute to make the artPiece done, after ending the auction of their art , the creators will split the ether share plus erc20 governance tokens. 

## Codebase Analysis 
###  a. Novel Areas 

1. **vrgda.sol** : this contract basically try to protect the **ERC20TokenEmitter** from the inflation and de-inflation of the tokens through adjusting the price of the token with respect of the time of buying. 
2. **maxHeap.sol** : the MaxHeap is a data Structure which is used to maintain the  maximum value in the top,the protocol uses it to keep the most voted art piece in the top .


###  b. Codebase Quality Analysis


1. The foundation of REVOLUTION's codebase is intricately organized, aligning with optimal practices in smart contract development. Employing a modular framework, with each feature implemented in separate contracts. This approach makes the codebase easier to navigate and understand.

2. The contracts are well-documented, with clear comments explaining the purpose and functionality of each function, but i have some notes:


    **a.** Writing the invariant is a positive indicator for the project and the team behind it.
    
    **b.** There is one word that confuses me while understanding the protocol: the word "creators." In some places, it refers to the creators of the art piece, while in other places, it refers to the founders of the protocol.


## Centralization Risks:

According to the RevolutionBuilder contract, all the **owner** and **manager** roles will refer to the DAO.

Therefore, the protocol is designed to be decentralized, with decision-making through the weight of the ERC721 (verbs token) and ERC20 token holders.

However, there are some risks from passing malicious proposals, such as changing the **reservePrice** in the **AuctionHouse.sol** contract simultaneously with an ongoing auction. It is better to have a minimal value for the **reservePrice** and to check it when changing the **reservePrice** value.

The same applies to other parameters such as **minBidIncrementPercentage** and **timeBuffer**.

## Recommendation:

1- Since the team is aware of the invariants of the protocol, formal verification for some scenarios or rules will be easier. I recommend formally verifying crucial contracts like maxHeap, AuctionHouse, and CultureIndex.

2- Since the protocol is similar to the Nouns project, the Revolution protocol shouldn't depend too much on the security process of the Nouns project. Every difference in one line of code may open the door to a new attack vector.


## Time Spent: 
~ 17 hours 




### Time spent:
40 hours