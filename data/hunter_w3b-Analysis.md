# Analysis - Revolution Contest

![Revolution-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FTBKrAmLWgoN.0&w=256&q=75)

## Description overview of The Revolution Contest

The Revolution Protocol aims to create a more equitable and participatory decentralized autonomous organization focused on community-driven creation and curation of creative works through weighted voting on submitted pieces, daily auctions of top ranked items with proceeds distributed to creators and protocol supporters, and an associated ERC20 governance token available for direct purchase through a continuous valuation model maintaining balanced distribution over time.

**The key contracts of Revolution protocol for this Audit are**:

Auditing the key contracts of the Revolution Protocol is essential to ensure the security of the protocol. Focusing on them first will provide a solid foundation for understanding the protocol's operation and how it manages user assets and stability. Here's why it's important to audit these specific contracts:

- **CultureIndex.sol**: This contract holds the community-submitted art pieces and implements the weighted voting mechanism. It's critical to ensure voting integrity and that submissions cannot be abused.

- **ERC20TokenEmitter.sol**: Minting and distribution of the governance token is central to the DAO model. It needs close examination to prevent exploitation of token issuance.

- **AuctionHouse.sol**: Auctioning the top pieces and distributing proceeds accordingly is a core economy driver. The flow of funds and permissions must be validated.

-- **VerbsToken.sol**: As the designated NFT for auctioned pieces, its functionality for minting from CultureIndex needs review.

Thoroughly auditing these four primary contracts that encompass art contribution, voting, tokenization, auctions and payments will uncover any vulnerabilities that could undermine the integrity of the Revolution Protocol.

## System Overview

### Scope

1. **Contracts**

- **MaxHeap.sol**: This contract implements a Max Heap data structure with functions for `insertion`, `updating values`, and `extracting the maximum element`. It enforces access control through an admin address, supports contract upgradeability via the UUPS pattern, and utilizes the OZ library and a custom interface for upgrade management.

- **CultureIndex.sol**: CultureIndex contract represents a decentralized culture index with voting functionality for art pieces. It features a max heap data structure for tracking top-voted pieces, supports ERC20 and ERC721 voting tokens, allows creation of art pieces with associated metadata and creators, and enforces various voting-related functions, including signature-based votes. Additionally, the contract incorporates upgradeability and access control, with an admin responsible for dropping new art pieces.

- **NontransferableERC20Votes.sol**: The NontransferableERC20Votes contract extends ERC-20 functionality with voting and delegation features, making the token non-transferable, and is designed for DAO governance, supporting delegation mechanisms and maintaining a history of vote power through checkpoints.

- **ERC20TokenEmitte.sol**: This contract facilitates the emission of a non-transferable ERC-20 token, integrating features like rewards, pausing, and split payments to treasury and creators, with customization options such as entropy rates and creator rates.

- **AuctionHouse.sol**: AuctionHouse designed to facilitate and manage auctions for Verbs ERC-721 tokens, with features such as auction creation, settlement, and bid handling, supporting ETH payments, and incorporating various configurable parameters for auction behavior, including time buffer, reserve price, minimum bid increment percentage, creator rates, and entropy rates, while also integrating with other contracts such as ERC-20 token emitter, WETH, and interfaces like IVerbsToken, IERC20TokenEmitter, IWETH, ICultureIndex, and IRevolutionBuilder, implementing OpenZeppelin libraries for upgradeability, pausability, and ownership control.

- **VerbsToken.sol**: This is an upgradeable implementation for managing Verbs NFTs, providing functionalities for minting, burning, and retrieving information about art pieces associated with each token ID, with features including a minter role, a token URI descriptor, and integration with CultureIndex for selecting art pieces.

- **VRGDAC.sol**: facilitates the sale of tokens based on a continuous issuance schedule. The pricing logic is governed by parameters including a `target price`, `per-time-unit price decay`, and a `decay` constant. The contract calculates the amount to pay `xToY` function and the number of tokens to sell `yToX` function based on the time since the auction start, the tokens sold, and the desired amount. The pricing logic uses a mathematical model that involves an integral of the price function, where the price decreases over time according to the specified parameters.

- **TokenEmitterRewards.sol**: The TokenEmitterRewards contract, inheriting from RewardSplits, manages token emission rewards with functions to handle reward distribution and value calculations based on specified conditions and parameters.

- **RewardSplits.sol**: The RewardSplits contract, defined with the MIT license, provides common logic for Revolution ERC20TokenEmitter contracts, facilitating protocol reward splits and deposits. It includes methods for computing total rewards, purchase rewards, and handling the deposit of rewards to various recipients based on specified settings and conditions. The contract incorporates constants and settings for reward distribution, and it uses an external interface, IRevolutionProtocolRewards, for depositing rewards.

### Protocol Roles

This `MaxHeap` contract has two roles:

1. **Admin:** The `admin` role is represented by the `admin` variable and is restricted to the contract owner. The admin role has the ability to insert new elements into the heap (`insert` function), update the value of existing elements (`updateValue` function), and extract the maximum element from the heap (`extractMax` function). The `onlyAdmin` modifier is used to restrict access to these functions to the admin role.

2. **Owner:** The contract owner has ownership-related privileges, such as upgrading the contract (`_authorizeUpgrade` function), initializing the contract during deployment (`initialize` function), and other ownership-related functionalities provided by the `Ownable2StepUpgradeable` base contract.

This `CultureIndex` contract has several roles:

1. **Owner:** The contract owner, who is initially set during deployment and is responsible for performing administrative functions. The owner role has access to upgrade the contract (`_authorizeUpgrade` function) and other ownership-related functionalities provided by the `Ownable2StepUpgradeable` base contract.

2. **Dropper Admin:** An address that is designated to have the privilege to drop new art pieces. This is set during contract initialization and can be used to execute the `dropTopVotedPiece` function.

3. **Voters:** Any address can act as a voter by calling the `vote` and related functions to cast votes for specific art pieces.

In this contract (`NontransferableERC20Votes`), there are two roles:

1. **Owner:** The contract owner, who has the ability to initialize and mint tokens. The owner role is established using the `Ownable2StepUpgradeable` base contract, and the `mint` function is restricted to the owner.

2. **Manager:** The contract upgrade manager, represented by the `IRevolutionBuilder` interface. The manager has the privilege to initialize the contract and is the only entity allowed to perform the initialization. This ensures that only the manager can set up the contract.

In this contract (`VerbsToken`), there are three roles:

1. **Owner:** The owner has certain privileged functions, such as pausing/unpausing the contract, setting parameters, and upgrading the contract.

2 **Revolution Builder:** The contract upgrade manager has exclusive initialization rights and ensures that upgrades are valid.

3. **Auctioneer:** The auctioneer initiates auctions, settles them.

## Approach Taken-in Evaluating The Revolution Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Revolution Protocol.
    I analyzed how these contracts work together to enable decentralized curation of art pieces, ERC20/ERC721 voting, daily auctions of top pieces distributed on-chain, revenue splits to creators and protocol, and emission of the REV governance token.

    The core contracts that implement the main functionality of the Revolution protocol including CultureIndex, MaxHeap, ERC20TokenEmitter, AuctionHouse, VerbsToken contracts.

    **Main Contracts I Looked At**

                CultureIndex.sol
                AuctionHouse.sol
                VerbsToken.sol
                MaxHeap.sol
                ERC20TokenEmitter.sol

    I started my analysis by examining the CultureIndex.sol contract, which serves as the main contract for curating and tracking art pieces on the Revolution protocol.

    The key functionalities it implements include:

    - Storing art piece metadata and associated creators
    - Tracking votes for pieces from ERC20 and ERC721 tokens
    - Maintaining a MaxHeap data structure to surface the most voted piece
    - Facilitating on-chain voting and gasless signature voting
    - Logic for creating new pieces, adding votes, and dropping the top piece
    - Configurable parameters like quorum voting thresholds

    Then, I turned my attention to the AuctionHouse.sol because it to be the contract responsible for managing Verbs DAO auctions. The AuctionHouse.sol contract facilitates the auction process for Verbs ERC-721 tokens, interacting with the VerbsToken contract to mint new tokens, manage bidding, and settle auctions.

    The Key functionalities it implements include:

    - The AuctionHouse interacts with the VerbsToken contract to mint new Verbs ERC-721 tokens and handle the transfer of tokens during auction settlements.
    - The contract has configurable parameters such as `timeBuffer`, `reservePrice`, `minBidIncrementPercentage`, `creatorRateBps`, `minCreatorRateBps`, `entropyRateBps`, and `duration`, allowing the DAO to adjust auction-related settings.
    - The AuctionHouse.sol contract provides functions to create new auctions, settle ongoing auctions, and handle the bidding process. The settleAuction function finalizes the auction, pays out to the winning bidder, and distributes funds to the creator and DAO based on configured rates.

    Then dive into the VerbsToken.sol contract because it serves as a key component in the Verbs DAO ecosystem, acting as the ERC-721 token responsible for representing unique cultural art pieces. The contract manages the minting and burning of Verbs while integrating with the CultureIndex contract to fetch top-voted art pieces. This ensures that each minted Verb corresponds to a culturally significant and community-approved piece of art.

2.  **Documentation Review**:

    Then went to Review [This Docs](https://github.com/collectivexyz/revolution/blob/main/docs.md) for a more detailed of Revolution protocol.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the contracts that are part of the Revolution protocol:

![Diagram](https://i.im.ge/2023/12/22/xrYhjr.chatuml-diagram.png)

- CultureIndex contract at the center storing art pieces
- Creators submit new pieces to CultureIndex
- VerbsToken contract minting ERC-721 tokens containing art metadata from CultureIndex
- AuctionHouse facilitating ERC-721 auctions
- ERC20TokenEmitter minting governance tokens sold via VRGDA
- Token holders using votes and bids via respective tokens
- Revenue split between creators and protocol via ERC20TokenEmitter
- Additional contracts for voting, rewards, upgrades

### Some potential areas for improvement and Architecture feedback

1. Support additional artist/creator integrations beyond single creator per piece

2. Formal Verification: Explore formal verification tools and techniques to prove the correctness of the contracts using a tool like Certora.

3. Reduce centralized dependency on ERC20TokenEmitter:

   - Explore alternative reward/governance models that don't rely solely on a centralized ERC20 token.
   - Allow for multiple emitting contracts controlled via DAO governance.

4. Revenue from daily Verbs auctions is split between creators and the DAO treasury/owner. This provides sustainable incentives for creators while also growing treasury funds.

5. Protocol rewards are allocated from token purchases to incentivize builders and contributors. Distributing a percentage of value as rewards aligns stakeholders.

## Codebase Quality

Overall, I consider the quality of the Revolution protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Revolution Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                       |
| **Code Comments**                        | During the audit of the Revolution contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary. |
| **Testing**                              | The audit scope of the contracts to be audited is 88% and it should be aimed to be 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. It inherits from multiple components, adhering for-example The CultureIndex contract follows a modular structure with clear separation of concerns into Core contract logic, inherited openzeppelin standards, and imported supporting contracts.Functions are grouped thematically and formatted uniformly with visibility, modifiers, and error handling.efficiency.                                                                                                                                                                                                                    |
| **Strengths**                            | Comprehensive unit tests,Utilization of Natspec                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Documentation**                        | Here are some suggestions for improving the documentation: 1. Add a high-level overview section at the top that briefly explains the purpose and goals of the protocol in 1-2 paragraphs for new readers. 2. Break out the information into clearer sections with headings (e.g. "Overview", "Core Contracts", "Creator Payments", etc). This makes it easier to scan and find relevant parts.3.Consider adding a brief overview of governance, tokenomics and economics (emission rates, incentives, etc).                                                                                                                                         |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Revolution protocol. These risks encompass concentration risk in SemiFungiblePositionManager, ERC1155Minimal risk and more, third-party dependency risk, and centralization risks arising. Additionally, the absence of fuzzing and invariant tests could also pose risks to the protocol’s security.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

2. The contract inherits from ERC20VotesUpgradeable but overrides the transfer-related functions to disallow transfers. This means that the tokens are non-transferable, and the associated voting power cannot be moved between addresses. While this might be a deliberate design choice, it restricts the usual functionality of ERC-20 tokens.

3. The ERC20TokenEmitter contract relies on startTime to calculate token emission. Any changes to the start time could affect the emission calculations and might introduce systemic risks.

4. In AuctionHouse.sol contract Auction parameters, such as the time buffer, reserve price, and minimum bid increment percentage, are initially set by the owner and can be changed later.Changes to these parameters can impact the fairness and security of the auction process.

5. The pricing logic in VRGDAC.sol contract involves complex mathematical operations, including exponentiation and logarithmic functions. Ensure that these calculations are gas-efficient and won't lead to unexpected computational costs, especially as the contract scales.

6. The VRGDAC.sol contract uses unchecked arithmetic operations, which means it assumes that overflow or underflow conditions won't occur. While this can optimize gas costs, it comes with the risk of unexpected behavior if arithmetic constraints are violated.

7. The TokenEmitterRewards contract checks if msgValue is less than the computed total reward before executing certain logic. It's important to ensure that such checks are accurate and that the contract handles ether values securely to prevent potential vulnerabilities like integer overflow.

### Centralization Risks:

1. CultureIndex::quorumVotes function the calculation of quorumVotes is based on a fixed percentage (quorumVotesBPS). A fixed quorum might be a potential centralization risk the total supply of tokens is controlled by a onlyOwner.

2. The ERC20TokenEmitter contract handles protocol rewards, and the distribution is controlled by the contract owner. This could introduce centralization, especially if the protocol rewards are not distributed fairly or transparently.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Revolution protocol.**

## Conclusion

In general, the Revolution project exhibits an interesting and well-developed architecture we believe the team has done a good job
regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from
potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance
understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security
measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
20 hours