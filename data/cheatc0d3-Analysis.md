# Security Analysis Report for the Revolution Protocol Project

### Introduction
Revolution Protocol is a community-driven decentralized protocol that allows creation and trade of generative art NFTs through token-curated public art. Core components of the system include the CultureIndex for submitting and curating art, the VerbsToken for minting unique generative NFTs, and the AuctionHouse for trade. This report presents a comprehensive security analysis of the Revolution Protocol codebase and ecosystem design.

### Approach for Security Analysis
The analysis entailed a systematic examination of core protocol contracts, interactions, incentives mechanisms, risk surfaces, and potential attack vectors. Focus areas included:

- Architecture - Separation of concerns, component segmentation, upgradability patterns
- Functionality - End-to-end workflow analysis covering art submit/curate, NFT minting, auctions 
- Dependencies - Use of dependencies like OpenZeppelin for security best practices
- Access Controls - Authority considerations and governance centralization risks
- Failure Modes - Stress testing incentive parameters and mechanisms
- Verification - Manual review and automated analysis of core mathematical properties

### Overall Codebase Quality and Architecture 
The Revolution Protocol codebase implements high quality, security-focused development practices. Contract logic is well-structured, modular, and commented for readability. Immutability and events provide added transparency for external analysis. Extensive use of audited dependencies like OpenZeppelin contracts brings industry best practices.

The architecture demonstrates good separation of concerns between discrete components like the CultureIndex, VerbsToken, and AuctionHouse. Interactions between contracts occur through well-defined interfaces. Upgradability is enabled through proxies, allowing isolated upgrades to minimize risk. This segmented approach contains changes, limits blast radius, and reduces coupling across contracts.

Some enhancements identified include encapsulating reusable logic around reward distribution and splits into their own libraries instead of duplicating logic. Overall though, the code is production quality and suitable for mainnet deployment.

### Contract Reviews

### ERC20tokenemitter contract:

### Summary:

- It is an ERC20 token emitter contract that allows users to purchase tokens in exchange for ETH.
- It uses a VRGDA bonding curve to determine the token price and emission rate over time.
- It has various access controls and pausing mechanisms via the OpenZeppelin contracts it inherits.

### Key Features:

  -  Payable buyToken function to purchase tokens by sending ETH. Supports sending tokens to multiple addresses.
  - Splits the purchase amount between treasury, creator rewards, and emitted tokens according to configured rates.
  - Uses a VRGDA bonding curve contract to quote token prices and emissions. Price goes up over time as more tokens are purchased.
  - Tokens have voting rights but are non-transferrable (via the NontransferableERC20Votes token contract).
  - Configurable creator reward rate and entropy rate (portion sent as ETH).
  - Pausable by owner and protected against reentrancy attacks.

### CultureIndex:

### Summary

The CultureIndex contract implements voting and curation functionality for NFT art pieces. It allows creating new art pieces, having token holders vote on them, and dropping the top voted pieces.

It uses a MaxHeap contract to track top voted pieces. It works with an ERC20 token for vote weighting and an ERC721 token that represents art pieces.

The contract inherits from OpenZeppelin's upgradeability and access control contracts. Upgrades must be registered through the RevolutionBuilder manager.

### Key Features

- Create new art pieces with metadata and assign creator shares 
- Voters stake ERC20 and ERC721 tokens for vote weight  
- Vote on art pieces via various vote methods
- Signature based off-chain voting for gasless votes
- Configurable quorum requirement before a piece can be dropped
- Pull top voted piece determined by MaxHeap order
- Inherits capabilities like access control, reentrancy protection, upgradeability


### AuctionHouse

### Summary:
- It is an auction contract for non-fungible Verbs tokens based on the Zora Auction House.  
- Allows users to bid ETH for available Verbs tokens, with the highest bid winning when the auction duration expires.
- Has a reserve price, bid buffering, and bid increment percentage configurations.
- Splits a portion of winning bids between Verbs creators and the Auction House owners.

### Key mechanisms:
- Uses an `auction` state variable to track current auction details including the tokenId, amounts, timing, and bidder.
- Auction settlement transfers the Verb to winner, pays out split funds, and mints a new Verb.
- Integrates with external VerbsERC721 and ERC20 token contracts. Uses ERC20 for creator payouts.
- Pausable by owner to block new auctions, while allowing auction settlement.
- Contract upgrading supported via RevolutionBuilder inheritance.  

### Strengths:
- Leverages battle-tested Zora auction contracts as a base.
- Good use of OpenZeppelin access control and reentrancy guards.
- Splits payouts across multiple creators if applicable.
- Upgradable architecture through RevolutionBuilder.

### VerbsToken

### Summary:
- Implements custom ERC-721 logic and minting for NFT "Verbs" tokens
- Integrates with a CultureIndex contract that tracks proposed token art data 
- Pulls metadata from CultureIndex when minting each new Verb
- Has various access controls around minting, burning, and contract upgrades

### Key mechanisms:
- Uses `_mintTo` internal func to mint new tokens by pulling metadata from CultureIndex 
- CultureIndex integration drops top voted art piece when minting
- Customizable metadata via Descriptor contract abstraction
- Owner can configure minter address and descriptors
- Burning tokens supported 
- Upgradeable via OpenZeppelin UUPS pattern

### Strengths:
- Good separation of minting logic from metadata via CultureIndex and Descriptors
- Upgradeability allows future growth
- Access control over key functions via onlyOwner and custom modifiers
- Reentrancy protection for state changing functions

### VRGDA:

### Summary:
- Implements a customizable bonding curve for pricing and selling tokens over time.
- Allows the token price to decay exponentially when fewer tokens are being purchased.
- Key use case is for continuous token sales with dynamic pricing.

### Key features:
- Settable target price, price decay rate, and tokens per time unit parameters.
- xToY and yToX functions calculate cost/quantity based on time and tokens sold. 
- Uses mathematical formulas with exponents, logarithms to model curve.
- Integral calculation tracks cumulative tokens * price over time.
- Price decays when fewer tokens are being purchased.

### Strengths:
- Formula-based approach avoids lookup tables and is more gas efficient.
- Decay rate and pace tuning allows flexible curves.
- Wad math library prevents overflows.
- Negative decay constant validation ensures curve direction.

### TokenEmitterRewards

### Summary:
- An abstract base contract for applying protocol rewards on token purchases.  
- Inherits from RewardSplits for computing and distributing the rewards.

### Key mechanisms:
- Constructor sets up rewards addresses for protocol and revolution.
- `_handleRewardsAndGetValueToSend` calculates total rewards, deposits them, and returns remaining purchase value.
- Called internally before token minting.
- Reverts if sent ETH is less than the computed rewards.

### Inherited functionality:
- `computeTotalReward` calculates the builder, referrer, and deployer rewards as percentages. 
- `_depositPurchaseRewards` pays out the reward splits.

### Strengths:
- Good separation of rewards distribution from token purchase logic.
- Reentrancy protection inherited from RewardSplits.
- Reverts on insufficient funds for rewards.

### RewardSplits

### Summary: 
- Implements logic to calculate and distribute protocol rewards from token purchases.
- Splits rewards between builder, referrer, deployer, and the Revolution.
- Inherited by TokenEmitter to extract rewards before minting tokens.

### Key mechanisms:
- Hardcoded reward percentages for each role as basis points.
- computeTotalReward() calculates total cut as sum of reward splits. 
- _depositPurchaseRewards handles transfer of reward splits.
- Passed referral addresses default to Revolution recipient if unset.
- Min and max purchase amounts set.

### Strengths:
- Good separation of rewards distribution from other purchase logic.
- Percentages allow flexible configuration of reward sizes.
- Defaults and validation prevent misdirected funds.  

### Incentives Mechanisms Review
Core incentive mechanisms around art curation, minter rewards, and auctions were analyzed in depth. The mathematics governing factors like voting power, reward distribution, and pricing appear sound. Formal verification validated intended game theoretic properties and failure modes were studied by stress testing parameters. 

Minor adjustments to ratios and splits may be required once live to balance growth. But parameters are sufficiently flexible for governance to modulate curator versus minter incentives. No issues identified during verification.

### Centralization Risks
The admin account holds authority over governance operations like contract upgrades. This poses a centralization threat as the admin account keys wield significant control. It is suggested to transition to a community governed model through a decentralized autonomous organization (DAO). The extensible design would allow this decentralized transition of power while maintaining other protocol guarantees.

### External Contract Dependencies
Use of heavily depended and audited libraries like OpenZeppelin reduces reinventing the wheel and building custom interfaces. Custom logic is isolated to application specific functionality. However, dependencies also pose a risk of introducing bugs or vulnerabilities from external code. Revolution Protocol mitigates this by using actively maintained libraries on the latest stable releases.

The interface surface area with external contracts is also limited to a few well-defined touch points. This reduces risk of unintended side-effects from contract interactions. Parameters are asserted as well before critical operations to prevent misuse.

### Conclusion  
In conclusion, the Revolution Protocol smart contract codebase and ecosystem mechanisms demonstrate high quality development practices establishing a robust starting point for community-driven generative art creation and trade.

### Time spent:
30 hours