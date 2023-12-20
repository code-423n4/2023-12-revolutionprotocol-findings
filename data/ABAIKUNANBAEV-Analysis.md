## Protocol overview

The purpose of the protocol is to make improvements on Nouns DAO making governance more accessible to users as builders will have an opportunity to create their own art piece and sell it instead of buying already generated tokenIds, as in the case with Nouns DAO. 

This includes several processes:

1. Creation: users create their own art pieces by providing necessary metadata and the creators array.
2. Voting: community members vote for the art piece they like and the best one is chosen by using max heap data structure.
3. Minting: the most voted art piece is minted as verbId.
3. Distribution: the newly created verbId pieces are distributed via auction procedure where everybody has a chance to buy them and become a community member with a right to vote.


![image](https://github.com/rodiontr/RevolutionProtocol/assets/109477234/ad6e0177-77f3-4ff5-933a-52cbfc3aa5ef)


## Main contracts flow overview

### CultureIndex.sol

![image](https://github.com/rodiontr/RevolutionProtocol/assets/109477234/d5fabf08-08e9-41f5-a388-e3651495ea3a)

`MaxHeap` contract is used to follow how many voting weight pieceId has and to dynamically change the positions of the corresponding elements if the voting weight changes to, finally, extract the root element. 

It's worth noticing that the contract has `dropperAdmin` which can only be `VerbsToken` contract that calls `dropTopVotedArtPiece()`.




### VerbsToken.sol

In this contract new verbIds are minted by the AuctionHouse (the minter role of the contract).

![image](https://github.com/rodiontr/RevolutionProtocol/assets/109477234/aef88c3f-1881-4117-899c-80579e29da62)




### AuctionHouse.sol

The contract serves as a place where verbIds are being distributed between the users via the auction procedure.

![image](https://github.com/rodiontr/RevolutionProtocol/assets/109477234/11476a87-5bef-40f0-ba57-1c50648c43e8)




### ERC20TokenEmitter.sol

![image](https://github.com/rodiontr/RevolutionProtocol/assets/109477234/4a511113-4a32-4215-9880-e177d7972574)



## Systemic risks

**Governance**: some of the governance design decisions can make the system work not as expected. For example, using `block.number` as the point where the users voting power is snapshoted instead of using current `block.timestamp` or another mechanism, is prone to creating various difficult situations where users don't mint anything because they wait for the more users to come into the system. This, again, happens because voting power is snapshoted and new vote pieces have an advantage over the older ones as more voters can vote for them. This problem is indicated in one of my findings.


## Centralization risks

**Centralized parameters**: the developers have an ability to set different parameters such as `creatorRateBps` and `entropyRateBps` (when initializing contracts of the system) that may affect user experience and the safety of their funds. It's important to make sure that all those variables have reasonable values. Moreover, only the owners may pause the contracts and change the state of the system.


## Security approach of the protocol

The protocol follows some of the concepts of Nouns DAO and improves them by making governance more accessible. The developers took into account past Nouns DAO audits to make sure that the same attack vectors don't apply to the system. It's also said that some private audits were conducted. However, it's highly important to audit contracts comprehensively to be confident that nothing is missed. It includes private audits, audit contests and audits from the firms. 


## Approach taken when evaluating the codebase

1. **Overview of the protocol**: at first stage, I looked into the protocol's contract very fast making some notes regarding of how the system is functioning overall.

2. **Deep dive**: after the quick look, I started to review contracts separately trying to form the clear picture in my head of how every action in the system happens and what are the main actors who initiate the system state changes.

3. **Running tests**: by setting up the testing environment, I made sure that all the tests passed and nothing remained uncovered.

4. **Complex attack vectors**: finally, after getting the full understanding of the protocol, I began to create some complex attack scenarios in my head trying to find new vulnerabilities.


## Conclusion 

In conclusion, the proposed protocol aims to enhance the governance model of Nouns DAO by allowing users to create and sell their own art pieces through a multi-step process involving creation, voting, minting, and distribution. 

The protocol introduces systemic risks, particularly in its governance design decisions. The use of block.number for snapshotting voting power may lead to unintended consequences, as users may delay minting, anticipating increased participation. This issue could potentially undermine the effectiveness of the governance mechanism.

Centralization risks are also present, as developers have the authority to set critical parameters like `creatorRateBps` and `entropyRateBps`, influencing user experience and fund safety. Furthermore, only contract owners can pause the system or alter its state, introducing a level of centralization that may raise concerns among users.

To mitigate potential security threats, the protocol incorporates concepts from Nouns DAO and claims to have undergone past audits, including private ones. However, it is worth mentioning that a comprehensive audit, encompassing private audits, contests, and assessments by reputable firms, is crucial to ensure the system's robustness.

The evaluation process involved an initial overview of the protocol, a deep dive into individual contracts, rigorous testing, and the exploration of complex attack vectors to identify vulnerabilities.

In moving forward, it is recommended to address the highlighted governance and centralization risks, conduct thorough and transparent audits, and consider potential improvements to the protocol's design to enhance its resilience and user trust.

### Time spent:
30 hours