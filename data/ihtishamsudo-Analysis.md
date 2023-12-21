## Revolution Protocol's Analysis Report 
### Preface 
This report is a summary of the analysis of Revolution Protocol's Codebase.
- This report is not an extension of the documents provided by Revolution Protocol.
- This report provides a high-level overview of the codebase and its security implications and some edge cases missed by developers and some suggestions to improve the codebase.
- This report aims to provide value to the sponsors and the developers of Revolution Protocol.

### Approach Taken Auditing Revolution Protocol
Day 1-3: 
- Read documentation provided by Revolution Protocol.
- Read the codebase and understand the flow of the codebase.
- Understand the architecture of the codebase.
- Evaluate the security implications of the codebase.
- Tried to expose some flaws of the codebase.

Day 4-5:
- Tried to find some edge cases missed by the developers.
- Started making notes with audit tags of the weak spots and potential flaw that can lead to vulnerabilities later on.
- Revolution Protocol is using ```transmission11``` VRGDAs, which is a new concept and is not often used by any other project. So, I had to understand the concept of ```transmission11``` VRGDAs and how they work.
- Checked out the difference between the actuall implementation of ```transmission11``` VRGDAs and the implementation of ```transmission11``` VRGDAs in Revolution Protocol.
- Thoroughly understood the codebase.

Day 6-7:
- Started making test cases for the notes made in the previous days.
- Filter out many false positives, and Low severity issues from H/M submissions.
- Submitted a potential vulnerability regarding signature replay which was one of the main invariant of the protocol as well as the main security feature of the protocol.
- Evaluate all the audit tags that were mostly Lows and Non Criticals then submit all issue in QA report alltogether.

Day 8 (Final Day):
- Submitted the final Analysis report with all the suggestions and process of the audit.
- Reviewed Again the overall codebase just to had a glance again to find out any potential flaw that I might have missed.

### Architectural Imporvement 
#### Protocol Strucutre 
Protocol Contains Two categories i.e, Core Contracts of Revoltuion Protocol and Reward Mehcainism Contracts. Overall Protocol is structured in a way that it is easy to understand and easy to maintain.
- ```protocol-rewards```
- ```Revolution```

![](https://gist.github.com/assets/82340173/c097d3af-28e0-453f-9ff2-3e97fc4083b4)

### What protocol Did Exceptional Things?


- Mentioning again about how exceptionally protocol used ````transmission11s``` VRGDAs to make the dynamic pricing so well maintained.
- Continuous Auctions is also the main feature of the protocol which is also implemented very well.
- More Engaiging Activities by the users so anyone can upload artpiece, it's pretty effective way to engage with your users and to make them intact with your protocol.
- Protocol using split method based on percentages to distribute the funds to the creators and the protocol itself. This is also a good way to distribute the funds.
- Non transferrable ERC 20 Token mainly used only for voting purposes and can only mint by the owner is also a good technique to make the protocol voting more secure.

### Things that can add up 

- Integrating ERC1155 tokens to not limit artist to ERC721 standard non-fungiblity model.
- Enabling delegatees to vote on specific art piece on behalf of delegator.
- Adding a feature to make the art piece transferrable.

### Codebase Quality Analysis

Codebase was written in well structure by rocketman and gpt4. Everything seems to be fair enough untill the common false assumptions made by the developers which can lead to potential vulnerabilities in the future pops up.

#### ```Rewards Contracts```: 
Rewards contracts were abstract contract and there was nothing much to look for. So, marking both these contracts are of high quality.

#### ```Revolution```:

- Culture Index : Culture Index is a contract that is used to store the art pieces uploaded by the users. It is a very important contract as it is used to store the art pieces and the art pieces are the main assets of the protocol. But it was not pretty well defined and it contains some of the flaws that can lead to DOS attack, submitting that issue in one of my report.

- Max Heap : This contract used to store to top vote piece by implementing binary tree data structure. The swapping of the nodes was somewhat fishy and weird as it could be done in a better way.
couldn't dig it well enough but it could cause some issue if put in arbitrary situation.

- AuctionHouse : It was a fork of nouns auction house. This is the contract that i find most opened to DOS attack. Used some bad practicies which can clearly leads to DOS attack. Also, It was a fork of noun auction house which had some flaws in the their recent audit done by an audit platform. So, it was a bad idea to fork a contract that is not audited well enough.

- VerbsToken : It was also a fork of nouns token. It had a flow through multiple contracts. i.e, VerbsToken -> CultureIndex -> Maxheap. Because Verbs Token is minted by Calling DropTopVotePiece and cultureIndex fetch dropVotepiece from MaxHeap. Was unfortunate to break that flow. Good practice by the developers and gpt4.

- ERC20TokenEmitter : This was the contract the was using VRGDAs. Although make notes about some attack ideas on this specific contract but was a good written contrac though.

- NonTransferableERC20Votes : Tokens that are used to vote on the art pieces. It was a good idea to make the token non transferrable. But it was not a good idea to make the token mintable by the owner. It could lead to some centralisation issues.

## Centralisation Risks 

There are trusted role in the protocol as (operatorsm slashers and pausers) which can lead to centralisation issues. Also, the owner of the protocol can mint the tokens which can also lead to centralisation issues.

- Centralisation that can brick the whole protocol : Because there was only one contract i.e, Revoltuion Builder that can initliaze and deploy almost all the contracts of the protocol. If that contract is compromised or had serious flaw that it can lead to the bricking of the whole protocol.

Overall, the codebase is not really centralised but used centralisation in such a way that can be dangerous for the whole protocol.

## High-Level Overview of the System 
[Gist For Relative Diagrams and Mental Models](https://gist.github.com/ihtisham-sudo/6bc09175e512f0eb8b31f243e7988f46)

How Protocol functions: 

![](https://gist.github.com/assets/82340173/ce5ff132-566a-4a9e-8eeb-7380538552a7)

## Chains Supported 

Ethereum Mainnet including Optimisim & Base As Well

## Features of the Main Contracts 

- Mental model of some main contracts is labelled in the below diagrams

![](https://gist.github.com/assets/82340173/26e9c8b3-98b9-419d-a134-187a34b9cad6)

## Systematic Risks : 

Using bad practices which can lead to the bricking of whole protocol as well as breaking of the Main Invariant of the protocol.

Anything that can be problematic to the protocol is the DOS attack. And Developers leaves many doors open for DOS attack. i.e, making external or public functions with having for loops with push mechanism. So any one can make that contract totally useless by just calling that function with a for loop and pushing the gas limit to the maximum.

As noted Above Depending on single contract to deploy all the contracts of the protocol can also lead to the bricking of the whole protocol. Which will make the protocol totally useless. Consider some backup or alternative in case ```revolutionbuilder``` contract is compormised.

Using OZ's EIP712Upgradeable and not implementing their recommendations of using ECDSA to get the signer address and using default ecrecover which can be lead to malleablitiy issues.

## Resource Consumed In Evaluting Codebase 

- Revoltution Protocol Docuemntation, again good work by the Team and gpt4.
- Some EIPs regarding the libraries which protocol inheriting 
- Fork Nouns previous audit of the contract that revolution protocol forked.

## Time Spent 

- 80 Hours of Audit


### Time spent:
79 hours