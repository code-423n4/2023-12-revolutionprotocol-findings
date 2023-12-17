Analysis Report :-

**Index of table**
| Sl.no  | Particulars  |
|---|---|
| 1   | Overview  |
| 2  | Architecture view(Diagram)  |
| 3  | Approach taken in evaluating the codebase  |
| 4  | Centralization risks  |
| 5 | Mechanism review  |
| 6  | Recommendation   |
| 7  | Hours spend  |


## 1. Overview

Revolution protocol aims to make governance token ownership more accessible by offering smaller denominations and alternative acquisition methods. This empowers creators and builders with a voice in shaping the project's future without needing to invest large sums  revolution protocol have a set of smart contracts built upon the foundation of Nouns DAO, aiming to address some perceived limitations and create a more accessible and balanced ecosystem for creators and builders. It goes beyond simply buying tokens. It explores alternative acquisition methods, such as contributing skills, completing tasks, or participating in community activities. This opens the door for creators and builders to earn governance rights through their valuable contributions, democratizing the power structure and ensuring a more diverse and engaged community.Instead of relying solely on expensive, whole tokens, it introduces smaller, more accessible denominations. This allows individuals to participate in governance and decision-making with a lower financial barrier. Imagine being able to buy a "fraction" of a governance token, granting you a proportionate voice in shaping the project's future.


## 2.  Architecture view(Diagram)

![FlowChart](https://user-images.githubusercontent.com/69415766/291059531-928c202c-5cd3-4b5e-b6a3-27715f082e49.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDI4MjEzODAsIm5iZiI6MTcwMjgyMTA4MCwicGF0aCI6Ii82OTQxNTc2Ni8yOTEwNTk1MzEtOTI4YzIwMmMtNWNkMy00YjVlLWI2YTMtMjc3MTVmMDgyZTQ5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMTclMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjE3VDEzNTEyMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTgwN2Q5ZGM2NmM5MjZiYWUwODEwMjRhOTdhNzEyMzEyZmFkMjFjMmI2MGQ0ZTlkNDk1MWJjNGE0ZjhmOWE4MTgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.LalANz0sZlxIyzjJm-K8DYUcnoh_Uq10UhNpHafOSpg)

Please Visit link to full view of diagram :-
https://lucid.app/lucidchart/8b9c4122-c00f-4d4b-9cd1-c828da2185fd/edit?beaconFlowId=4457157C27F2E5DF&invitationId=inv_1bc46c7c-2b69-4f2c-853e-8c1198064da2&page=0_0#

or 
http://icp-tool.rf.gd/Revolution.png


## 3. Approach taken in evaluating the codebase 
We approached manual code review by looking into each space in the scoped contract and decided to explain some core functional contracts.it is important to note that manual code reviews can be very time-consuming.

The main contracts are :-
1.MaxHeap.sol
2.CultureIndex.sol
3.VerbsToken.sol
4.AuctionHouse.sol
5.VRGDAC.sol
6.ERC20TokenEmitter.sol
7.Rewards.sol

**1.MaxHeap.sol**
This contract plays vital role in revloution protocol by implementation of Heap and Max Heap data structure and it functionalites in placing the top voted art piece in parent node and less voted art piece as the child node of the root node(parent node). This eliminates the traditional voting system and effecient in nature also.The MaxHeap contract represents a significant leap forward in voting system design within the Revolution Protocol. Its innovative application of data structures not only streamlines the voting process but also fosters a more dynamic, efficient, and transparent experience for all participantSs.

State variables are :-
`address admin` to stores the admin adress.
`heap` mapping to store the structure of the heap
`uint256 size` to stores the size of the heap.
`valueMapping` mapping to store value of an item in the heap.
`positionMapping` mapping to store the position of the item in the heap.

Core functions are:-
`parent()`
This function, calculates the index of the parent node for a given position in a Max Heap data structure. The pos - 1 part accounts for the indexing starting from 0, while the / 2 part performs the floor division to find the parent index based on the binary tree structure of a Max Heap. In a Max Heap, the left child of a node is at 2pos + 1 and the right child is at `2pos + 2`, so the parent is located at floor((pos - 1) / 2).

`swap()`
This function efficiently swaps two elements in a heap and updates the corresponding position mapping to ensure accurate tracking of element locations.

`maxHeapify()`
This function, maxHeapify, maintains the max-heap property of a data structure called a "heap". It takes a starting position, pos, and ensures that the element at that position has a value greater than or equal to its child node. It finds the positions of the left and right child node of the element at `pos`. Then it retrieves the values of the element at pos and its child node using a mapping, `valueMapping`, which links heap positions to their actual value and avoids unnecessary processing if the element is already in the lower half of the heap finally if the element's value is less than either child's value, it needs to be heapified if the left child node has a greater value than the right, the element is swapped with the left child node and `maxHeapify` is called again on the left child's new position.Otherwise, the element is swapped with the right child and maxHeapify is called on the right child node's new position.

`insert()`
This function takes two arguments an item ID and its value. It inserts the item ID into the heap mapping and updates the corresponding value and position mappings. The function then walks up the heap, swapping the item with its parent if the parent's value is less than the child's. This process continues until the inserted item reaches its appropriate position in the heap, maintaining the heap property where each parent's value is greater than or equal to its child node's values. Finally, the function increments the heap size to reflect the new item.

`updateValue()`
This function  updating the value of an existing item in the heap while maintaining its max-heap property. It takes the two argmuents item ID and its new value as inputs. First, it finds the item's position and its original value in the mappings `postionMapping` and `valueMapping`. Then, it updates the value in the mapping. Based on whether the new value is greater or smaller than the old one, it performs either upwards or downwards heapify. If the new value is larger, it iteratively swaps the item with its parent until it reaches its rightful position in the heap, ensuring parent values remain greater than child values. If the new value is smaller, it directly calls the `maxHeapify()` function to adjust the heap downwards, again upholding the max-heap property. This function essentially ensures the heap remains properly structured after updating an item's value.


**2. CultureIndex.sol**
This smart contract is a platform where creators and artists can add new art pieces. These pieces can then be voted on and auctioned by holders of non-transferable ERC-20 voting tokens. This contract plays a vital role in facilitating economic activities for the Revolution protocol. Which is intialised by AuctionHouse contract.

Core function are:-

`createPiece()`
This function is responsible for creating new art pieces in the CultureIndex contract. It takes two arguments: ArtPieceMetadata containing details about the artwork and CreatorBps specifying the percentage distribution of token voting power among creators.First, it performs a series of validations on both arguments to ensure they comply with specified rules. Then, it initializes a new ArtPiece variable (newPiece) and assigns values from the provided arguments and internal calculations.

`vote()`
This function functions as an interface for holders of non-transferable ERC-20 tokens to vote for art pieces and access information about top and least-voted pieces. It performs extensive validation on the provided arguments to ensure data integrity.To cast a vote, the function retrieves the voter's weight via the getPastVotes() function. It then updates the piece's vote count and the total weighted votes, reflecting the new vote. Finally, it calls the updateValue() function to adjust the piece's position within the internal heap data structure based on its updated vote count.

`dropTopVotedPiece()`
This function is used to pulls the top voted Art piece in-order to auction-off. First it checks whether the `msg.sender` is `dropperAdmin` or not then calls `getTopVotedPiece()` function to get the piece information and checks the whether the piece quorum values meets or not finally assigns `isDropped` bool value to true it means it cannot be called again. And calls
`extractMax()` fucntion to reduce the heap structure.

**3. VerbsTokens.sol**
VerbsTokens play a crucial role in creating new auctions within the Revolution Protocol. Each auction is backed by a minted ERC-721 token representing the top-voted art piece that has been simultaneously dropped from the CultureIndex contract.


`mintTo()`
This function, likely called by the AuctionHouse contract when creating a new auction, mints a new ERC-721 token based on a verbId. It performs several validations on the input arguments before attempting to `dropTopVotedPiece` from the CultureIndex contract. This function returns the verbId if successful, but fails with an error message if the dropping process fails. But in this try and catch blocks it copy the ArtPiece to memory which is not used anywhere in through out life cycle. Which will cost execution gas.

**4. AuctionHouse.sol**
This contract, AuctionHouse.sol, plays a critical role in the Revolution Protocol by acting as the central hub for managing art piece auctions. It facilitates the creation of new auctions for top-voted art pieces from the CultureIndex. This involves minting corresponding ERC-721 tokens as representations of the auctioned pieces.Users can interact with existing auctions by placing. The contract tracks all bids, ensuring fair competition and transparent bidding history.Once an auction concludes, the contract determines the winning bid and facilitates the transfer of ownership of the associated art piece (represented by the ERC-721 token) to the winner. It also distributes rewards and fees in accordance with the predefined auction parameters.

Core functionalities are :-
`createBid()`
This function allows users to place bids on existing auctions with amounts exceeding the current highest bid. It implements a multi-step validation process to ensure data integrity and fair bidding practices. Firstly, it verifies that the provided arguments adhere to specified rules. Then, it checks if the sender's bid `msg.value` is greater than the current highest bid. If so, it updates the auction's highest bidder to the sender `msg.sender`. Additionally, if the time remaining in the auction is less than a predefined 'timeBuffer', the function automatically extends the auction duration to prevent last-minute sniping attempts. Finally, it refunds the previously highest bidder their deposited amount.

`_createAuction()`
This function facilitates the creation of a new auction following the conclusion of the previous one. Initially, it performs a crucial gas check to ensure sufficient resources for subsequent operations.If sufficient gas is available, the function calls the mint() function of the VerbsToken contract to mint a new ERC-721 token representing the next art piece up for auction. Finally, it assigns a specific value to the newly minted token based on pre-defined rules or factors related to the art piece's characteristics or the platform's economic model.

`_settleAuction()`
Sure, here is a summary of the function in a paragraph:
This function is called internally to settle an auction after it has ended. It first checks to make sure that the auction has not already been settled and that the current time is past the end time of the auction. If all of the checks are passed, the function sets the settled variable of the auction to true to indicate that it has been settled.Next, the function checks to see if the contract has enough money to cover the reserve price of the auction. If it does not, then the function refunds the last bidder and burns the verb. Otherwise, the function checks to see if anyone bid on the auction. If no one bid, then the function burns the Verb. If someone did bid, then the function transfers the Verb to the winning bidder.The function then calculates how much money should go to the owner of the auction, the creators of the verb, and the treasury. The amount of value is then transferred to the appropriate addresses.Finally, the function emits an AuctionSettled event to let everyone know that the auction has been settled.

**5.VRGDAC.sol**
This contract plays vital role in revolution protocol by implementing the continuous variable rate gradual dutch auction (VRGDA) mechanism for selling tokens.It aims to mimic a pre-defined issuance schedule for token distribution, adjusting the price dynamically based on time and the number of tokens sold. It helps to determine how much the price decreases with each unit of time without any sales.The price declines gradually over time, following an exponential decay curve. This contract facilitates the controlled sale of tokens according to a predetermined schedule, adjusting the price to incentivize purchase while ensuring gradual distribution and preventing rapid dumping. This mechanism helps regulate token supply and market stability within the Revolution Protocol ecosystem.

Core function are :-
`pIntegral()`
This pIntegral function in the VRGDAC contract calculates the total amount of amount would be pay for a specific number of tokens `sold` based on the current time `timeSinceStart` and the total number of tokens already sold. It essentially integrates the price curve of the VRGDA over the specified amount of sold tokens.

`xToy()`
This function calculates the amount of amount need to pay `y` to purchase a specific number of tokens `amount` in the VRGDAC at a given point in time.

`yToX()`
This function performs the inverse calculation compared to xToY().It determines how many tokens  can get `x` for a given amount of money `y` at a specific point in time.It performs a complex calculation involving exponents, logarithms, and multiplication to determine the number of tokens corresponding to the provided amount. This calculation essentially inverts the price curve from `pIntegral()` and takes into account the current time, remaining supply, target price, price decay, and the provided amount.


**6.ERC20TokenEmitter.sol**
This contract plays a crucial role in the Revolution Protocol by enabling the continuous linear purchase of its ERC20 governance token. It allows anyone can purchase the ERC20 token at any time with a predictable price curve.Tokens backed by this contract ar non-transferable erc20 tokens called as governance tokens which will be needed to voting an art piece.

Core function are :-
`buyToken()`
This function allows users to purchase tokens for themselves and other addresses. The purchase amount is split based on percentages specified in basisPointSplits. A portion of the amount goes to the protocol rewards, treasury, and creators, while the rest is used to mint tokens (non-transferable erc20 tokens) for the buyers and creators. The function ensures that everyone involved receives their fair share and keeps track of the total tokens emitted.

**7.RewardSplits**
This is an abstract contract that implements a mechanism for calculating how rewards should be split between builders and creators, both during the purchase of governance tokens and after the settlement of an auction.

core functions are :-
`computeTotalReward()`
this function calculates the total rewards distributed during a token purchase. It takes the purchase amount  as input and verifies it falls within the allowed range. Then, it applies pre-defined percentages `basis points` to the amount to determine individual reward shares for builders, purchase referral source, deployer, and the revolution itself.

`_depositPurchaseRewards()`
This function used to determines rewards earned during a token purchase first calls computePurchaseRewards to determine the total reward amount and individual reward shares for different stakeholders based on predefined percentages.If any of the provided referral or deployer addresses are empty, it sets them to a designated `revolution reward recipient` address.Finally, it calls the `protocolRewards.depositRewards()` function to deposit the calculated rewards to the corresponding  builder, purchase referral, deployer, and the revolution itself. Each recipient receives their designated share as specified in the RewardsSettings structure.

`computePurchaseRewards()`
This function calculates and packages reward details for a token purchase by calculating individual reward shares for builder referral, purchase referral, deployer, and revolution by applying pre-defined percentages `basis points` to the purchase amount. These are stored in a RewardsSettings structure.


## 4. Centralization risks
The function listed below have `onlyOwner()` modifier will landed on risk if owner address compromised by various factors.
[`setCreatorRateBps()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L217)
[`setMinCreatorRateBps()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L233)
[`setEntropyRateBps()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L253)
[`setMinBidIncrementPercentage()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L297)
[`burn()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L184)
[`setMinter()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L209)
[`lockCultureIndex()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L262)



## 5.  Mechanism review 

After manual testing, we concluded that a revolution protocol reimagines Nouns protocol's model, empowering creators and builders through affordable governance tokens. It seeks to bridge the chasm between artistic expression and economic value, while guaranteeing the continual expansion of decision-making power within the community. This innovative marketplace serves as a platform for artists to unleash their creative talents, with community members casting their votes to choose the most deserving piece. The artwork garnering the highest vote is then auctioned off, with a portion of the winning bid distributed among the artist, builders, and the auction contract owner. A further share is allocated to the artist in the form of non-transferable voting tokens, granting them a voice in shaping the future of the platform. This unique system empowers creators to not only reap financial rewards but also actively participate in shaping the direction of Revolution DAO.

**Comparision between Revolution protocol and Noun DAO**
| Feature  | Revolution DAO  | Nouns DAO  |
|---|---|---|
| Focus  | Empowering creators and builders  | Collective ownership and treasury growth  |
| Governance Tokens  | smaller denominations, alternative acquisition methods  | Single denomination, auction-based acquisition  |
| Treasury Allocation  | Split between creators, builders, and auction contract owner, with portion to creators in non-transferable voting tokens  | 100% to treasury  |
| Decision-making Power  | Continuously expanding through governance token inflation  | Concentrated among early investors and large token holders  |
| Revenue Streams  | Schedueled auctions, alternative initiatives  | Daily auctions  |
| Benefits for Creators  | Direct funding through grants and dedicated treasury allocation, non-transferable voting tokens for influence  | Indirect benefit through potential treasury use  |
| Accessibility  | More accessible governance participation for creators and builders  | More accessible governance participation for creators and builders  |
| Long-term Sustainability  | Focus on diverse revenue streams and inflation-based governance growth  | Reliance on daily auction revenue  |
| Overall Vision  | Bridge the gap between artistic expression and economic value, empower creators and builders  |  Decentralized ownership, collective decision-making, treasury growth |

## 6. Recommendation

After manual observation of code and mechanism we recommend a simple changes which make more robustness to revolution protocol.

1. Add function mechanism to set the dropper admin.
Implement a function to designate the Dropper Admin . This is crucial because the current Dropper Admin being the Verbs Token Contract poses a risk. If the Verbs Token Contract were compromised, it could be upgraded independently, potentially jeopardizing the top-voted piece and causing losses for creators. By adding a mechanism to change the Dropper address, even in the event of a compromise, we can safeguard the top-voted piece. Alternatively, we could implement a pause and unpause functionality, by implementing the below code snippet allowing the top piece to be auctioned even if the Verbs Token Contract is compromised.

```solidity
function setDropperAdmin(address newDropper) onlyManager {
	dropperAdmin = newDropper;
}
```

2. Add the pause and unpause mechanism for some crucial functions also which would be helpfull in the time of any financial risk or security risks.
Below can be adopt the mechanism :-
[`Vote()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L332)
[`dropTopVotedPiece()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L519)
[`createPiece()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L209)
[`createBid()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L171)

3. There is chance if overflow errors in [`_calculateVoteWeight()`]() function depending on the data types used for erc20Balance and erc721Balance, the multiplication with erc721VotingTokenWeight * 1e18 could potentially overflow and lead to incorrect results. It's essential to choose appropriate data types that can handle the expected range of values.Add some division operation will help to get the precision value.

4. Rounding Down to Zero issue :-
We can take look into the below varaibles and function first .

```solidity
    uint256 internal constant DEPLOYER_REWARD_BPS = 25;
    uint256 internal constant REVOLUTION_REWARD_BPS = 75;
    uint256 internal constant BUILDER_REWARD_BPS = 100;
    uint256 internal constant PURCHASE_REFERRAL_BPS = 50;
    uint256 public constant minPurchaseAmount = 0.0000001 ether;
    uint256 public constant maxPurchaseAmount = 50_000 ether;
    
    function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

        return
            (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000 + (paymentAmountWei * PURCHASE_REFERRAL_BPS) /10_000
             +
            (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000 + (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000;
    }
    
    
```

The value of `paymentAmountWei` variable of the function argument could be 0.0000001 and more than that makes the return variable as zero only as we know solidity doesn't support decimals point add some check that if the calculated return is less than the minimum, we can set the return value to the minimum instead. This ensures that even small payments contribute to the system.

5. Add mechanism to check the total supply of tokens is greater than zero or minimum amount before calculating vote weight of a voter or cultureIndex contract.

6. Currently, Noun DAO auctions operate sequentially, with each new auction beginning only after the previous one concludes. Revolution Protocol can proposes new idea to modify this traditional approach by enabling the simultaneous auctioning of multiple verbs. This could be achieved by replacing the singular auction house contract with a data structure like a mapping, associating verbs with their respective auctioning process information.

### Time spent:
45 hours