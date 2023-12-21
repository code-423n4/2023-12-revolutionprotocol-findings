### Comprehensive Project Overview

Revolution is a protocol that improves on Nouns DAO - a generative avatar collective that auctions off one ERC721, every day, forever. 100% of the funds from each auction go into a shared DAO treasury, and owning an NFT gives you 1 vote over the treasury. 

The Revolution twist is that, instead of auctioning off a generative NFT like nouns, anyone can upload their art pieces, and the community votes on their favorites.

The auction proceeds are split between the art creator and the owner of the auction contract:

- The creator of the art piece receives ERC20 governance tokens and a share of the winning bid
- Auction owner is transferred the remaining ETH from the winning bid
- The winner of the auction(the highest bidder) receives the ERC712 art piece

### Main Contracts Analysis

***1. AuctionHouse.sol:***

Purpose: AuctionHouse.sol facilitates the sale of ERC721 tokens, specifically VerbsTokens. Forked from NounsAuctionHouse, it ensures fair compensation for creators by splitting auction proceeds.

Key functionality: 

- `_createAuction`**:** Initiates the creation of an auction by attempting to mint a VerbsToken. It checks for sufficient gas before executing the minting process, sets up the auction parameters such as start and end times, and emits an **`AuctionCreated`** event if successful. If the minting operation fails, the function pauses the contract.
- `settleAuction`: Finalizes an auction, paying out to the owner, creator(s), and the winning bidder based on predefined rates. If there are no bids and the reserve price isn't met, the Verb is burned. In case of a successful auction, the winning bidder receives the Verb, the owner gets a portion of the bid, and the creators are rewarded with both ether and ERC20 governance tokens according to the defined rates.
- `createBid`: Enables users to place ETH bids on specific Verb auctions, validating bid conditions, including the bidder and auction status. It updates auction details, extends if needed, and refunds the previous bidder


***2. ERC20TokenEmmiter.sol:***
Purpose: the contract is a linear [VRGDA](https://www.paradigm.xyz/2022/08/vrgda) that mints an ERC20 token when the payable **buyToken** function is called, and enables anyone to purchase the ERC20 governance token at any time. A portion of value spent on buying the ERC20 tokens is paid to creators and to a protocol rewards contract.

Key functionality: 

- `buyToken`: The **`buyToken`** function facilitates the purchase of tokens using ETH. It validates the sender and payment amount, distributing the ETH between treasury and creators based on predefined rates. It calculates tokens for both buyers and creators, transferring these tokens accordingly. Ensures the correctness of the basis point splits and returns the total tokens allocated to buyers.

**3. VerbsToken.sol:**

Purpose: fork of the [NounsToken](https://github.com/nounsDAO/nouns-monorepo/blob/master/packages/nouns-contracts/contracts/NounsToken.sol) contract. **VerbsToken** owns the **CultureIndex**. When calling **mint()** on the **VerbsToken**, the contract calls **dropTopVotedPiece** on **CultureIndex**, and creates an ERC721 with metadata based on the dropped art piece data from the **CultureIndex**.

Key functionality: 

- `mint`/ `burn` - minting and burning of Verbs tokens

**4. CultureIndex.sol:**

Purpose: CultureIndex.sol serves as a repository for uploaded art pieces, allowing anyone to contribute media. Users who own ERC721 or ERC20 tokens can vote on art pieces, with their voting power determined by their token balances. The art piece voting data is managed using MaxHeap.sol, a heap data structure that facilitates efficient retrieval of the highest-voted art piece. The contract includes a dropTopVotedPiece function, which removes the top-voted item from the MaxHeap and returns it.

Key functionality: 

- `createPiece`: generates a new art piece with specified metadata and a list of contributing creators, each assigned a basis point percentage. The function validates the creator array, ensuring it does not contain zero addresses and that the sum of basis points is exactly 10,000. The created art piece is assigned a unique ID, added to a max heap for efficient voting, and relevant events are emitted, including details about the piece, its sponsors, and the contributing creators.
- `vote`: allows a voter to cast a vote for a specific ArtPiece. It checks for the validity of the piece ID, ensures the voter's address is valid, and verifies that the voter has not already voted for the given piece. The function checks that the piece has not been dropped, and the voter's weight exceeds the minimum vote weight. Once validated, the vote is recorded, the total vote weight for the piece is updated, and relevant events, including details about the vote and total weight, are emitted.
- `voteforManyWithSig`: allows a voter to execute multiple votes using a signature. It first verifies the validity of the signature and upon successful verification, the function proceeds to cast votes for the specified list of **`pieceIds`** on behalf of the specified voter.
- `dropTopVotedPiece`: allows the designated dropper admin to pull and drop the top-voted piece from the culture index. It verifies that the caller is the designated admin and checks if the top-voted piece meets the required quorum votes to be dropped. If the conditions are met, it marks the piece as dropped, removes it from the heap, and emits a **`PieceDropped`** event, returning the details of the dropped piece.

**5. MaxHeap.sol:**

Purpose: Implements a MaxHeap data structure. More about the structure can be read here: https://www.geeksforgeeks.org/introduction-to-max-heap-data-structure/

**6. NontransferableERC20Votes.sol:**

Purpose: Simple nontransferable ERC20 token that represents votes in the system

**7. VRGDAC.sol:**

Purpose: Implementation of a VRGDA(Variable Rate Gradual Dutch Auction) used by the `ERC20TokenEmitter` to emit ERC20 tokens at a predictable rate. The VRGDA contract dynamically adjusts the price of a token to adhere to a specific issuance schedule. If the emission is ahead of schedule, the price increases exponentially. If it is behind schedule, the price of each token decreases by some constant decay rate.

**8. TokenEmitterRewards.sol:**

Purpose: abstract contract that extends the **`RewardSplits`** contract. It's designed to compute and handle rewards and deposits for the TokenEmitter

**9. RewardSplits.sol:**

Purpose: The contract is responsible for calculating and distributing rewards based on different parameters like builder referral, purchase referral, deployer rewards, and revolution rewards. These are represented as basis points (BPS) and are used to calculate the respective shares of the total reward.

### Architecture/Contract Interactions
Here is a basic diagram of how the core contracts of the system interact with each other:
![RevolutionArchitecture](https://img001.prntscr.com/file/img001/9ksx3pWLQVSFa2siX7a9yQ.png)
Link if the image doesn't work: https://img001.prntscr.com/file/img001/9ksx3pWLQVSFa2siX7a9yQ.png

### Centralization & Systemic risks
Here is a list of the governor roles and what rights they have for each contract.

In **AuctionHouse**:

Owner - can pause/unpause, `setMinCreatorRateBps`, `setEntropyRateBps`, `setTimeBuffer`, `setReservePrice`, and authorize upgrades via `_authorizeUpgrade`

In **ERC20TokenEmmiter**:

Owner - can pause/unpause, `setEntropyRateBps`, `setCreatorRateBps`, and `setCreatorsAddress`

In **VerbsToken**:

Owner - can `setContractURIHash`, `setMinter`, `lockMinter`, `setDescriptor`, `lockDescriptor`, `setCultureIndex`, `lockCultureIndex`, and `_authorizeUpgrade`

In **CultureIndex**:

Owner - can `_setQuorumVotesBPS`, and `_authorizeUpgrade` 

In **NontransferableERC20Votes**:

Owner - can `mint` an unlimited amount of tokens

In **MaxHeap**:

Owner - can `_authorizeUpgrade`

**Centralization risks**:

- Token Concentration: If a small group of participants holds a significant amount of tokens, they could have a disproportionate influence on decision-making and governance.
- The owner can mint an unlimited amount of NontransferableERC20Votes tokens 

- The owner can change the `CreatorRateBps`, `MinCreatorRateBps`, and `CreatorsAddress` which means that in case the owner gets compromised or is malicious, he can direct a big % of the auction proceeds to addresses of his own.

### Recommendations
Overall the codebase is relatively simple and of good quality, the functions have implemented proper access control, and are interacting well with each other. 
A 100% test coverage should be achieved.

### Approach taken
Day 1-3:
- Understand the idea of the project and grasp a high-level mental model of the contract interactions
- Review the contracts and take notes

Day 3-6:
- Deep dive into each contract
- Following user flows
- Running tests, tweaking them for edge cases and testing if everything works correctly

Day 6-8:
- Submitting issues and Analysis
- Having a last thorough look over the contracts

### Conclusion 
In conclusion, the Revolution protocol represents a significant evolution in the realm of decentralized autonomous organizations (DAOs), building upon the foundational concepts established by Nouns DAO. By auctioning community-voted art pieces and directly compensating creators, Revolution introduces a more inclusive and dynamic approach to governance and token distribution.

Key features of the protocol include:

- The ability for any participant to upload art to the CultureIndex, democratizing the process of content selection.
- The use of ERC721 VerbsTokens and ERC20 governance tokens to give voting power to a broader community base, enhancing the inclusivity of decision-making.
- Innovative mechanisms for reward distribution, including the splitting of auction proceeds between creators and the auction contract owner, and the linear VRGDA system for ERC20 token emission.

The project's aim to balance cultural contributions with capital influence and to make governance token ownership more accessible aligns with a growing trend in the blockchain community towards more equitable and participatory models.

Overall, the Revolution protocol embodies a progressive step in the evolution of DAOs, focusing on fair ownership distribution and community empowerment.

### Time spent:
20 hours