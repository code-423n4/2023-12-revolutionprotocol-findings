
![Revd](https://github.com/code-423n4/2023-10-ethena/assets/125544245/e7e39a4e-03f3-4455-a210-5916477e36d1)

Revolution is an innovative protocol seeking to empower communities to collectively fundraise, distribute governance rights, and maximize social impact. 

It builds on the NounsDAO model but aims to improve fairness through tokenized voting and rewards for creators. At its core is a cultural index for community artwork, dynamic NFT auctions, and layered token governance.

## **Analysis Approach**

My analysis evaluated both the systemic security and technical implementation of Revolution. This included:

**Architectural Review**

### **Token Distribution Model**

Revolution has an innovative dual token model for fundraising and governance:

**Auction NFTs (VerbsTokens)** [**VerbsTokens**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol)

- Daily minted ERC-721 NFTs representing top community artwork
- Minted by pulling metadata from [**CultureIndex**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol) 
- Auctioned via [**AuctionHouse**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol) contract
- Primary source of fundraising

**Governance Tokens**

- [**Non-transferable ERC-20**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol) minted by [**ERC20TokenEmitter**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol)
- Minted on purchases or from [**AuctionHouse**](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol) creator proceeds  
- Balance confers voting power over artistic submissions
- Future community governance capabilities 

This dual token approach separates financial from governance value.

### **Auction and Artwork Flow**

The artwork path from submission to NFT auction is:

1. Artists upload metadata to CultureIndex
2. Token holders vote on art based on ERC-20 and ERC-721 balances
3. Top voted metadata used to mint new VerbsToken ERC-721
4. AuctionHouse runs daily NFT auction 
5. Settlement splits proceeds between creators and treasury

This leverages the CultureIndex as a democratic art surface and funding source.

### **Creator Payment Structures**

Revolution's dual creator payment pathways are:  

**Auction Proceeds**
- Configurable `creatorRateBps` portion of auction settlements
- Split into direct ETH payments and governance token grants

**ERC20 Purchases**
- Configurable `creatorRateBps` portion of token purchases
- Split into direct ETH payments and remainder sent to configured address

This incentivizes creation of artwork and protocol participation.

## **Code Quality**

### **Adherence to Best Practices**

Revolution contracts follow several security best practices:

- Inheriting core logic from **battle-tested OpenZeppelin contracts** like Ownable, ERC20, AccessControl which have been rigorously audited
- **Modular design** separating key functionality into distinct contracts with clean interfaces
- **Explicit visibility declarations** on functions and state variables
- **Emitting custom events** for critical state changes
- **Input validation** in external functions

This indicates general awareness around secure development patterns. Additional rigor is needed but the foundation is decent.

### **Presence of Security Checks**  

Some key security checks are present including:

- **Access control** restricting privilege using onlyRole modifiers to trusted roles like owner and minter 
- **Input validation** enforcing invariants like total creator basis points summing to 10k
- **Reentrancy guards** preventing reentrant calls

But other best practice checks are missing:

- Lack of **numeric overflow checks** on auction payment math and other sensitive calculations
- No **maximum numeric bounds** enforced on inputs like array lengths

More comprehensive security hardening is required to avoid vulnerabilities.

### **Documentation Quality**

- Helpful **docstrings** explain and document expected behavior of public/external functions
- Code **comments** call out assumptions, dev notes, and future protocol improvements
- **Natspec** tags describe intended purpose of state variables

Overall documentation quality aids auditability but specifics around edge cases and security would further improve understanding.

## **Risk Analysis**  

### **Centralization Risks**

The **RevolutionBuilder** contract concentrates privilege over:

- Initializing core protocol contracts
- Managing upgrades of those contracts

> This creates a central point of failure. If the owner key is compromised, the attacker gains tremendous power over auction setup, token distribution, etc.

They could for example reinitialize contracts to extract funds or modify logic.

**Mitigations**

Grant the `RevolutionBuilder` admin role to a DAO multi-sig to decentralize power.

### **Privilege Concerns** 

The **NFT minter** holds power over:
  
- Daily minting of new NFTs to auction 
- Burning NFTs

If this key is compromised, the attacker could disrupt NFT auctions - a key revenue stream.

They could mint NFTs sending funds to an address they control for example. 

**Mitigations**

Use a time-delayed DAO-controlled minter address.  

### **Economic Sustainability**

The **VRGDA emission schedule** for governance tokens seeks to balance supply with demand. 

If token demand falls significantly below expectations, extreme token oversupply could create extreme centralization pressure and essentially valueless governance power.

For example, if purchases dry up but auctions continue, few entities would end up controlling governance.

**Mitigations**

Carefully tune parameters to sustainable levels. Have programmatic ability to quickly adjust if needed.

## **Key Findings**

### Positive Observations

- **Thoughtful and fair token model** - Sensible approach with auction NFTs + non-transferable governance token distribution to creators and community.

- **Code modularization** - Each primary component is sensibly separated into discrete contracts with clean interfaces.

- **Care around upgrades** - Use of UUPS proxy pattern with RevolutionBuilder managing process.

### **Areas for Improvement** 

- **Centralization risks** - RevolutionBuilder controls critical privilege over initializations and upgrades. Use of a DAO-controlled admin could decentralize this.

- **Unchecked math vulnerabilities** - Several locations enabling manipulation of auction proceeds, governance totals. SafeMath style checks could mitigate.

- **Potential auction DoS vectors **- Array lengths like creator lists should be bounded to prevent settlement blocking.

### **Positive Observations**

**Thoughtful Token Distribution Model**

The dual token structure with auction NFTs and governance tokens enables novel separation of financial and governance value. This helps distribute power more widely to creators and participants beyond just wealthy NFT collectors.

Overall, the model comes across as more egalitarian within its domain compared to precedents like NounsDAO.

**Code Modularization**

Each key protocol component like the CultureIndex, AuctionHouse, ERC20TokenEmitter, etc is sensibly separated into its own discrete contract. This improves legibility, maintainability and simplicity compared to a single monolithic contract.
    
Interfaces between contracts are well-defined leading to a modular and extensible architecture.

**Careful Handling of Upgrades**

Use of the UUPS proxy pattern isolates upgrade risk to minimized code churn. And involvement of the RevolutionBuilder admin provides a checkpoint protecting against supply manipulation in upgrades.

This reduces potential attack surface from malicious upgrades while still keeping core logic flexible.

### **Areas for Improvement**   

**Centralization Around RevolutionBuilder** 

The RevolutionBuilder admin holds concentrated privilege to initialize contracts and control upgrades. Compromise of this single key would enable replacing core protocol logic or extracting funds.

Ideally this role would be decentralized to a DAO multisig over time to sufficiently disperse trust.

**Unchecked Math vulnerabilities**

Several areas like the AuctionHouse payment divisions and VRGDAC pricing math lack numeric overflow protections. This could allow manipulation of auction proceeds or token totals.

Addition of Safemath style overflow checks before division/multiplication arithmetic would help mitigate these risk vectors.

**Potential DoS Vectors**

Unbounded creator arrays or voting batches create risk of gas limit blocking settlement transactions or bloating storage.

Enforcing max numeric constraints on key array lengths would close these potential denial of service vectors. 

## **Recommendations**

1. Decentralize RevolutionBuilder admin to a DAO multisig 
   - Reduces dependence on single owner key

2. Apply circuit breaker / maximum bounds wherever appropriate
   - Prevents business logic blocking from unbounded operations

3. Introduce pre-transaction state validation  
   - Detect manipulation attempts on governance totals
   
4. Consider sustainability planning and incentives
   - Encourage continual community involvement 
  

### Time spent:
29 hours