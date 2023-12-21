## **Summary**

Revolution is an ambitious protocol seeking to empower participatory community funding and governance. It builds on the NounsDAO model but distributes power and rewards more widely via an artistic CulturalIndex, daily NFT auctions, and layered token voting.
  
## **Analysis Approach** 

My in-depth analysis evaluated:

- **Architectural soundness** - Token model, auction flow, payment structures
- **Code quality** - Best practices, documentation, security checks
- **Risk factors** - Centralization, sustainability, fail-safes

I utilized both manual review and static analysis tools to methodically evaluate the system.

## **Key Findings**

### **Positive Attributes**

- **Thoughtful dual token model** balancing financial and governance value  

  **Separating Financial and Governance Rights** 

Unlike most crypto protocols which consolidate power into a single token, Revolution purposefully separates out financial from governance value accross two distinct tokens:

**Auction NFT Token (VerbsToken)**

- Daily minted as ERC-721 representing top community art piece 
- Primary source of community fundraising via auctions
- Ownership confers no special governance privileges 

**Governance Token**

- Non-transferable ERC-20 token emitted by ERC20TokenEmitter
- Earned by auction creators and ETH purchases 
- Balance proxies voting influence over artistic submissions
- Future protocol governance powers

This structure ensures governance input can come from both financially invested auction participants as well as engaged community contributors without capital.

For example, an active artist could earn governance tokens enabling voices in decisions but without winning an expensive NFT auction.

**Broader Governance Distribution**

Unlike most NFT platforms where influence ties directly to wealth, Revolution's split token model supports wider governance distribution not just the financial elite.

It embraces more horizontal participation across both capital and creative classes.

- **Modular contract structure** separating key logic into discrete components

**Decoupling Key Functions into Contracts**

Rather than a single monolithic contract, Revolution splits functionality out into separate first-class contracts including:

- [**CultureIndex**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol) - Artwork submission and voting
- [**AuctionHouse**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol) - Daily NFT auctions
- [**VerbsToken**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol) - ERC-721 NFT minting
- [**ERC20TokenEmitter**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol) - Governance token distribution 

This reflects separation of concerns and creates more maintainable and extensible code.

**Improved Readability and Flexibility**

With modularization, each component contract focuses on one primary purpose rather than a tangled weave of logic jumping between functions.

For example, auction bid placement and finalization logic lives solely within the AuctionHouse. Metadata highlighting resides strictly within CultureIndex.

This simplifies understanding of the isolated moving pieces and their interactions.

Upgrading any singular component also becomes easier with less risk of side effects. We can validate in isolation rather than re-verifying global code integrity across broad logic.  

**Well-Defined Interfaces** 

Rather than direct integration, contracts interact through clearly defined interfaces. For example:

- AuctionHouse relies on IVerbToken(VerbsToken) for NFT minting  
- VerbsToken calls into ICultureIndex(CultureIndex) to fetch metadata

This abstracts sub-system complexities away from callers.

Overall the modularization scheme aids security, auditability, separations of concern, and accelerates development velocity.

- **Careful upgrade methodology** using UUPS proxy pattern

**Isolating Upgrade Risks** 

Rather than relying on direct contract state, core Revolution logic like the [AuctionHouse](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol) and [CultureIndex](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol) are deployed via UUPS proxy contracts. 

This enables owning only the tiny proxy logic rather than entire heavy contract codebases.

**Upgrading via UUPS**

To upgrade Revolution contracts:

1. New logic contract is deployed
2. Proxy admin calls `upgradeTo` on proxy, pointing to new logic address
3. Proxy delegates all functions calls to new logic contract 

This means only the proxy admin and tiny delegatecall surface needs reviewing for upgrades rather than re-verifying integrity of entire codebase.

**RevolutionBuilder Authorization** 

The RevolutionBuilder contract manages access over initializing new contract implementations and upgrading existing proxy contracts.

Before any upgrade, the RevolutionBuilder checks that the new implementation has been registered as an authorized upgrade. This provides an important checkpoint protecting overall supply and integrity.

**Limiting Surface Area**

By isolating upgrades down to a single administrative contract,  the potential attack surface is greatly minimized compared to relying on individual owner keys or multi-phase proposals.

This helps mitigate risks around malicious or defective upgrades while keeping core protocol logic flexible.

### **Areas for Improvement**

- **Single point of failure** around RevolutionBuilder owner admin key  

**Concentrated Control in RevolutionBuilder**

The RevolutionBuilder contract manages critical functionality like:

- Initializing core protocol contracts
- Controlling authorized contract upgrades

This key coordinating role currently rests in the hands of the singular RevolutionBuilder admin.

**Security Risks**

If compromised, the RevolutionBuilder admin has tremendous power including:

- Reinitializing contracts to transfer funds
- Pointing to malicious contract logic via upgrades
- Approving defective or exploitative upgrades

Attackers could leverage this to siphon community funds, modify rules, or break auctions.

**Centralized Trust** 

Currently, all protocol participants must inherently trust the motivations and security posture of the lone RevolutionBuilder admin. This creates centralization risk and tensions around perceived fairness and censorship.

**Mitigations** 

To reduce this centralized trust, the admin role can migrate to a smart contract or DAO multisig over time. This distributes trust across multiple parties to better align with community interests versus a single key.

- **Unchecked math vulnerabilities** enabling manipulation of auction proceeds and governance totals

**Lack of Overflow Protection in Calculations**

Several key Revolution contracts perform mathematical calculations on uint or int variables without overflow protection:

**AuctionHouse**

Settlement logic splits proceeds between creator shares and treasury via division

**ERC20TokenEmitter**

VRGDAC pricing functions calculate token costs and supplies using unprotected math:

**Exploiting Overflows**

By passing extremely large inputs, attackers could trigger overflows and manipulate outputs.

For example, bidding 2**256 - 1 ETH could allow greatly boosting creator share percentage in AuctionHouse settlement.

This violates expected business logic and proportional splits.

**Mitigation** 

Use OpenZeppelin's SafeMath library to automatically check for overflows before math.

- **Potential denial-of-service vectors** from unbounded operations

**Uncapped Array Lengths Risk Causing Gas Limit Errors**

The Revolution CultureIndex and AuctionHouse allow users to submit array-based data like:

- AuctionHouse - Array of `creators` in artwork metadata
- CultureIndex - Array of `pieceIds` voted on 

However, there are no limits enforced on the lengths of these arrays.

**Potential for Gas Limit Exhaustion**

Without limits, attackers could submit arrays with extremely large lengths like 2**256. 

When processed, these lengthy arrays could cause out-of-gas errors halting transactions.

For example:

```
CultureIndex.voteForMany([gigantic creator array])
// Runs out of gas due to array length
```

**Denial of Service Risk**

Since operations like auction settlement and voting are non-reentrant, these out-of-gas halts could permanently lock core functionality.

Attackers could essentially DoS the ability to finalize auctions or vote on submissions.

**Mitigation** 

Enforcing maximum lengths on unbounded arrays helps prevent this attack vector.

## **Recommendations**

**1. Mitigate centralization risks**

- Decentralize RevolutionBuilder admin to DAO multisig over time
- Apply Circuit breaker / maximum bounds where possible
- Pre-transaction validation of governance totals

**2. Add overflow & underflow protections** 

- Use OpenZeppelin SafeMath for uint operations 
- Explicitly check bounds on key math integrations

**3. Introduce sustainability planning** 

- Model token emissions dynamically
- Monitor, incentivize community involvement

### Time spent:
34 hours