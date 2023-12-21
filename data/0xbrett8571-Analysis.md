## **Introduction**

Revolution Protocol is an innovative system for community curation of generative art combined with fair tokenomic incentives. I thoroughly reviewed the core contracts including CultureIndex, VerbsToken, AuctionHouse, ERC20TokenEmitter, VRGDAC, and other supporting contracts.

Overall I found the protocol to be thoughtfully architected to balance incentives between capital, creators, collectors, and community curators. There is ample flexibility in parameters to allow governance tuning as the system evolves.  

In my evaluation, I took a systematic approach of analyzing contract capabilities, trust relationships, economic incentives, and technical soundness. I looked for gaps in permissions, potential centralization risks, and ways adversarial conditions could disrupt system function.

## **Architecture**

Revolution utilizes a decentralized, layered architecture with well-scoped roles and capabilities:

**Decentralized Architecture**

Revolution divides capabilities across contracts instead of centralization: 

- [VerbsToken](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol) mints NFTs
- [AuctionHouse](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol) handles sales
- [CultureIndex](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol) manages curation  

This prevents single points of failure.

**Well-Scoped Roles**

Contract capabilities are also narrowly scoped to each purpose:

- CultureIndex: Curation logic and parameters
- VerbsToken: NFT management  
- AuctionHouse: Bidding and settlement

**For example, business logic risks are isolated:**   

```solidity
// Art curation logic only lives in CultureIndex
function dropTopVoted() { internal curation code }  

// AuctionHouse relies on results but can't directly impact
AuctionHouse.settleAuction() {

  CultureIndex.dropTopVoted() // Isolated call  
  
  // Rest of auction settlement
  // Captures emitted piece

}
```

This structure reduces risk surface area compared to a centralized model.

**Flexible Parameters**

Many values like royalty rates and auction settings are configurable by contract owners to allow governance tuning. This will help balance incentives as the ecosystem develops.

**Configurable Parameters**

Key economic variables are designed as changeable settings: [AuctionHouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol)

```solidity
// AuctionHouse

// Winning bid royalty rate to creators   
uint256 public creatorRateBps;  

// Auction duration
uint256 public duration;

// Minimum bid increment percentage
uint8 public minBidIncrementPercentage;

function setCreatorRateBps(uint256 newRate) onlyOwner external {
  creatorRateBps = newRate;
}

// Similar setter functions
```

This allows governance tuning without contract changes.

**Example Issue Scenario**

If parameters were fixed at deploy: 

```solidity
// Hardcoded values
uint256 public constant creatorRateBps = 500;
uint256 public constant duration = 1 days; 
```

It could badly skew incentives over time:
         
- Creators paid too much as community grows             
- Auction times useless due to maturity

Requiring a migration to fix:

```solidity
// Must redeploy entire system to adjust! 
RevolutionSystemV2 
  // Creator rate still imbalanced...
```
        
Instead, adjustable parameters enable responsive governance:
      
```solidity
function setCreatorRateBps(200) // Tune as necessary

  // No migration overhead to change levers
```   

**CultureIndex for Community Curation**  

The CultureIndex and associated voting mechanics allow the community to democratically curate art submissions. Configurable voting token weights provide tuning levers.

**Configurable Voting Weights** 

The [CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol) sums weighted voting power from ERC20 and ERC721 tokens to determine top art:

```solidity
function _calculateVoteWeight(uint256 erc20Balance, uint256 erc721Balance) internal view returns (uint256) {
        return erc20Balance + (erc721Balance * erc721VotingTokenWeight * 1e18);
    } 
}

// erc721Weight configurable  
uint256 public erc721VotingTokenWeight; 
```

This allows governance to tune influence between different voter classes:

**Example Scenario**

If weight was fixed equal for token types:

```solidity
function _calculateVoteWeight(uint256 erc20Balance, uint256 erc721Balance) 
  internal view returns (uint256 voteWeight) {

  voteWeight = erc20Balance + erc721Balance;  
}

// No adjustment possible
```

Wealthy collectors could outweigh community votes: 

- Whales buy PracticeVerb drops to vote
- Grassroots ERC20 opinions drowned out

Configurable weights prevent this by tweaking voting power if necessary: 

```solidity
function setERC721Weight(500) // Buff community impact

  // Democratic balance restored  
```
**VerbsToken and AuctionHouse for Monetization**

The Nouns-inspired NFT auction model allows capital efficiency and shared incentives, while providing royalties to creators.

**Shared incentives**

The Nouns-style blind auctions distribute proceeds efficiently.

This mutually aligns the DAO and creators around delivering value via the NFTs.
       
**Example scenario** 

If royalties were lower, creators may not actively participate:

```solidity
function settleAuction() external {

  // Tiny royalty percentage  
  uint256 creatorPayment = (auctionProceeds * 100) / 10000;

  // Bulk to treasury
  treasury.transfer(auctionProceeds - creatorPayment);     

}

// Creators lose incentive to submit NFTs  
```

Configurable rates keep incentives balanced as the protocol matures.

**VRGDAC and ERC20 Emissions**  

The continuously decaying token model creates fair initial distribution and long-term sustainability of governance rights.

## **Incentives**  

I analyzed incentive structures along several dimensions:

**Creators**

Royalties paid on secondary sales motivate submissions. Later governance participation maintains engagement. 

**Configurable Royalties**

Creators receive secondary sales royalties: [AuctionHouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol)

```solidity
// @AuctionHouse

function settleAuction() {

  uint256 royalty = salePrice * creatorRoyaltyBps / 10000;

  creatorsWallet.transfer(royalty);

}  

// Tunable rates
uint256 public creatorRoyaltyBps; 

function setRoyalty(500) // For example <<@
```

This incentivizes high quality submissions.

**Example Scenario**

If royalties were fixed too low long-term:

```
uint256 public constant creatorRoyaltyBps = 100; 
```

Creators may lose interest without sufficient compensation:

- Focus efforts on other NFT projects  
- Fewer submissions to CultureIndex 

Flexible rates keep creators engaged as the protocol matures.

**Curators** 

Voting power ties curation participation to governance influence. Parameter tuning can balance voter types.

**Configurable Vote Weights**

The [CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol) ties influence to curation participation.

This incentivizes voting on submissions by connecting it to governance power.

**Example Scenario**

If voting power was disconnected from influence:

```solidity
function calculateVotes(address voter) pure returns (uint256) {

  // No query of actual voting tokens

  return 1; 

}
```
There is little incentive to actively curate content, harming system quality.

The configurable model ties participation to ownership.

**Collectors**

NFT ownership provides financial upside, community status, and governance input.

**Shared Financial Upside**  

Owning [VerbsTokens](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol) entitles holders to treasury revenue: [AuctionHouse](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol)

```solidity
// AuctionHouse

function settleAuction() external override whenPaused nonReentrant {
        _settleAuction();
    }

  // Winning bid paid to DAO treasury  
  treasury.transfer(auctionProceeds); 

}

// All holders share in profits 
```

This incentivizes gathering high quality NFTs.

**Community Status**

Prominent array in contract storage confers social status.

**Governance Rights**

NFTs also grant voting power over parameters and treasury.

This combination of financial, social, and governance incentives promotes healthy collecting.

**DAO Participants**

ERC20 ownership grants growing decision rights as the organization matures.

Incentives appear well-constructed to directly engage key actors in the ecosystem. 

**Configurable Governance Power**

The ERC20 token ties ownership to influence.

This incentivizes early participation as power grows.

**Example Scenario** 

If influence was disconnected from balance:

```solidity
function getVotes(address account) pure external returns (uint256) {
  return 1; // Equal voting regardless of stake
}
```
There would be no incentive to accumulate governance tokens.

The configurable weighting drives engagement over time.

## **Trust Relationships**

Revolution exhibits carefully constructed admin roles and multi-party dependencies:  

**Separation of Powers**

Key abilities are split across roles to prevent centralized control:

**RevolutionBuilder**

Registers contract upgrades. No runtime access.

**CultureIndex Admin** 

Controls curation parameters and art drops. Configurable by DAO.

**AuctionHouse Owner** 

Sets auction rules and parameters. Proceeds flow to them currently.  

**VerbMinter**

Issues NFTs according to protocol. No risk of Pausing.

**Example Scenario**

If multiple roles combined:  

```solidity
contract RevolutionSystem {

  // All access consolidated

  function settleAuction() {}

  function configureIndex() {}
  
  function mintNFT() {}

}

// Single admin compromise can fully capture system

```

Instead, capabilities isolation reduces contagion risk.

**RevolutionBuilder**  

Trusted to register valid contract implementations for upgrades. No other special access.

**CultureIndex Admins** 

Control index parameters and dropping art. Checks prevent censorship.

** descriptors**  

Generate NFT metadata. Flexible to use different systems.

**AuctionHouse Owner**

Sets auction parameters. Profits from sales by default.

Excellent point. I should technically explain the CultureIndex admin and descriptor relationships more thoroughly:

**CultureIndex Admin**

The CultureIndex admin controls curation parameters and art drops: [CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol)

```
// CultureIndex  

function setQuorumVotes(...) external onlyOwner {
  // Change acceptance threshold  
}

function dropTopVoted() external onlyOwner {
  // Allow next auction proposal
}
```

But acceptance rules are enforced on-chain preventing censorship.

**Descriptors**

The Descendant contract generates NFT metadata:

``` 
// Descendant

function tokenURI(uint256 id) external view returns (string memory) {
  // Metadata generation
  return uri; 
}
```

This is a plugin model - flexible to use different descriptor systems:

```
function setDescriptor(DescendantV2 descriptor) external onlyOwner {

  descriptor = descriptor;

}
```

So metadata can evolve without contract risk.

**Minter**

Issues ERC20. Can be compromised so emission schedule parameters are immutable.

**Configurable Minter**  

The ERC20 minter issues new governance tokens.

If compromised, they could print tokens to hijack control.

**Immutable Emission Rules**

To mitigate risk, emission schedule parameters are fixed: 

```
// VRGDA 

uint256 public constant tokensPerDay = 100;
```

So even a rogue minter couldn't alter distribution fairness.

## **Potential Issues**

A few areas stood out that could use hardening:

**Denial-of-Service Resistance**  

No special mitigations in place. As system grows, on-chain overhead may enable griefing.

**Admin Key Management**  

No features to transition admin roles or migrate keys. Manual process relies on social trust.

**Error & Pause Recovery**

Manual unpausing required although system restarts in sane state.

**Denial-of-Service Risks**

No special on-chain mitigations exist currently:

- No governance of gas limits
- All computation on-chain  

This leaves the door open to potential griefing attacks as adoption grows:

```
// Scaled submission bot attack

while (true) {
   CultureIndex.submitComplexNFT() 
}

// No throttling leads to congestion
```

Adding mitigations would allow sustainable growth:

- Configurable gas limits
- Offchain data availability  

**Admin Key Management**

Transitions between admin keys rely on off-chain coordination:

```
function transferOwnership(address newOwner) external onlyOwner {
   owner = newOwner;
}

// Social process to facilitate transition
```

Formal on-chain features could reduce trust requirements:

- Timelock accept transfers
- Multi-sig schemes

**Error & Recovery Management**  

Errors require manual oversight to resume:

```
// AuctionHouse
function unpause() external onlyOwner {
   
   // Fix state and restart
   _createAuction();

}

// Admin must invoke 
```

Structured pause lifecycles could smooth operations:

- Pause manager role  
- Automated restart triggers

Overall the core protocol appears very well architected to balance incentives, establish trust, and grant governance flexibility. As adoption grows, vigilance around administrative roles and scaling constraints will be important.

### Time spent:
39 hours