# **1. Codebase Reveiew**
## **1.1 General Review**

- Nouns is a generative NFT project on Ethereum, where a new Noun is minted and auctioned off every day, all proceeds go to the NounsDAO treasury which governa and control the actions of the Noun protocol, and each token represents one vote.
- The Revolution protocol aims to improve on this method by making the governance structure more equitable and democratic - making the governance tokens more accessible to any and all interested users, not just the creators and builders.
- The protocol incentivizes community-driven art curation in which anyone can upload and vote any artpiece of their choosing, using the protocol's erc20/erc721 votes tokens. Topvoted art pieces are converted into a unique verbs nft and are auctioned. 
- It also incentives profit sharing as art creators and the platform share the consequent auction proceeds, granting voting tokens to the creators and the auction buys in process.
- For the purpose of this audit, Revolution Protocol consists of nine smart contracts totaling 1000 SLoC. Its core design principle is inheritance, enabling efficient and flexible integration. It utilizes four external calls to facilitate communication with other blockchain components. It is planned to be deployed on Ethereum, optimism and base chains.
- The protocol exists as an upgrade of the Nouns Dao and is expected to comply with the ERC721 and EIP 20 token standards. It operates independent of oracles and sidechains but does use a continuous Variable Rate Gradual Dutch Auction function forked from Paradigm.

## **1.2 Scope and Architecture Overview**


### **1.2.1 Contracts**

#### **Voting contracts**
- [CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol) - is the contract users interact with to upload new pieces of art. Community members can then vote on these pieces, and the topvoted piece if it meets quorum is sent off to be minted for auction. The uploaded artpieces are stored in the MaxHeap contract.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/292120986-13490738-eb5a-41e0-8c4b-c5d18a06aa16.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231221%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231221T071159Z&X-Amz-Expires=300&X-Amz-Signature=16a38ec28746db51a7c3f792f7f95d5aab86c3da984b95394fa74b33877260e2&X-Amz-SignedHeaders=host&actor_id=112232336&key_id=0&repo_id=732778523" alt="CultureIndex">
</p>



#### **Auction contracts**
- [AuctionHouse.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol) - receives the minted NFt of the topvoted artpiece from the VerbsToken contract and puts it up for auction. Here, users can place bids on artpieces of their choice, and at the end of the auction, proceeds are split between the art creators and auction owner based on a predefined creator and entropy rate. If needed, it makes calls to the `ERC20TokenEmitter` to mint the ERC20 votes tokens for the creators. The artpiece NFT is then sent to the highest bidder. 
<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/292127715-349e4440-c805-42f5-97cf-ee0c0a7b82f4.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231221%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231221T073805Z&X-Amz-Expires=300&X-Amz-Signature=67167af1f0c66a3ee4351229cd35b719042baef3b12df8de1393ee1d266be1aa&X-Amz-SignedHeaders=host&actor_id=112232336&key_id=0&repo_id=732778523" alt="!AuctionHouse">
</p>

#### **Minter contracts**
- [VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol) - is the main minter contract of the protocol. When a new auction is to be created, a call from the AuctionHouse to this contract is redirected to the CUltureIndex to retrieve the topvoted artpiece. It then mints an NFT of the artpiece, before putting it up for auction in the AuctionHouse. Verbs that didn't get sold are also burned here.
<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/292136315-329bb0f6-b6ca-493a-9021-a8cdf54f6d66.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231221%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231221T080823Z&X-Amz-Expires=300&X-Amz-Signature=2f1e1e962533116d8762a483a6b8f0a0332509c2e8fd05d0302b0adea6a93273&X-Amz-SignedHeaders=host&actor_id=112232336&key_id=0&repo_id=732778523" alt="VerbsToken">
</p>

- [ERC20TokenEmitter.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol) - mints the `NontransferableERC20Votes` tokens for parties interested in purchasing, and for art piece creators if needed. A portion of the amount spent on the tokens is paid to the contract defined creator addresses and protocol rewards contracts, depending also on predefined rates. 

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/292143250-70185ab8-1b13-4252-a20a-00bbee7fa1d1.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231221%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231221T082740Z&X-Amz-Expires=300&X-Amz-Signature=a2ab601b0071788dd3c7c102cb14ed9aa054156c56c059af4a0ec5bd3a302718&X-Amz-SignedHeaders=host&actor_id=112232336&key_id=0&repo_id=732778523" alt="ERC20TokenEmitter">
</p>

#### **Utility contracts**
- [MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol) - stores the data of votes cast on artpieces in the Culture index. It uses a Tree-based heap datastructure, in which the tree is a complete binary tree. The data at the top of the tree represents the topvoted piece. When the topvoted piece is called for auction, the contract removes its data from the heap. 

- [NontransferableERC20Votes.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol) - is an extension of ERC20 votes token that cannot be transferred. Users can buy these tokens through the ERC20TokenEmitter. If needed, it can also be minted to art piece creators when their artpiece gets auctioned.

- [VRGDAC.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol) - The continuous linear Variable Rate Gradual Dutch Auction used for NontransferableERC20Votes token emission.  It allows the NontransferableERC20Votes toekns to be issued with almost any schedule while allowing users to buy them at any time. It works by raising prices when sales are ahead of schedule and lowering prices when sales are behind schedule. 

- [TokenEmitterRewards.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol) - Compute rewards and deposit for the ERC20TokenEmitter.

- [RewardSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol) - Compute and deposit rewards based on creatorrate and entropyrate splits.

## **2. Risks to protocol**

- **Centralization** - The protocol is controlled by the DAO using VerbsDAOLogicV1 contract which owns all of the major contracts. Major protcol implementations, proposals are made by the DAO, which, while not being in scope of audit, is assumed to be well centralized. The voting process in the CultureIndex also aims to foster decentralization. However, its important to note that there's no veto power available, which leaves the protocol vulnerable to a 51% attack on the voting process. Beyond thos, decisions made by the DAO affect the protocol in a major way. For instance, the reserve rate could be set so high that all verbs eventually be burned due to bidders not willing to pay that much for them. 

- **MultiChain risks** - The contracts are to be deployed on Ethereum, Optimism and Base chains, which have slightly different opcodes. For instance, the compiler version in use, `0.8.22` is not supported on L2 chains, consequntly contract deployment on Optimism and Base will fail. In the same vein, the use of block.number to get past votes is not very reliable on L2 chains. This is better replaced with block.timestamp parameter which is more accurate. Seeing that users are allowed to vote using signatures, there's a risk of cross-chain replay attack, especially as the user signatures do not implement a chainId parameter. Consider mitigating these to reduce these risks.

- **Inter contract dependencies** - The protocol consists of a number of contracts interacting with each other, hence a problem in one, could flow into the other and pose a risk to the protocol's functionality. One of these possible cases is the dependency of the AuctionHouse on the ERC20Tokenemitter. Whether the auction house is paused or not, auctions need to be settled, however, if the ERC20Tokenemitter which can also be paused, gets paused, settling auctions becomes impossible. As a consequence, new auctions can't be created and a core part of the protocol's process is stopped. The same goes for other contracts too, a simple math error (probable as division before multiplication precision error can occur) in the VRGDAC could have a ripple effect on the protocol's functionality. 

- **ThirdParty dependencies** - The contracts imports OZ contracts and its upgradable counterparts, (version 5.0.0 is being used) and solady's `signedWadMath` contracts. While these have been well vetted, its important to note that no contract is fully airtight. Likewise, using the latest versions of these dependencies is not really recommended as they may contain yet to be discovered vulnerabilities. Consider switching to more proven and tested albeit older versions.

- **Risks from upgrades** - The contracts are designed to be upgradable, with the revolution builer authorized to perform these updgrades. Upgrading a smart contract is not very easy and needs to be handled carefully to prevent something from breaking. Seeing that the contracts lack a timelock, every new implementation takes place immediately. This should be implemented, to give time to reset things, in case of a compromise.
- **Other external factors** - DAO hijack, scams artists, inside hacks, social engineering attack, etc can also in their own ways affect the protocols.

## **3. Audit approach**
We approached the audit in 3 general steps after which we generated our report.

 - **Documentation review** - We reviewed the readMe, documentation and explanations provided by the devs on discord. While this was going on, we ran the contracts through slither and compared the generated reports to the bot report.

- **Manual code review** - Here, we manually reviewed the codebase, ran provided tests, tested out various attack vectors. We looked for ways to DOS the system, ways a user can game the system, while ensuring that the contract comply to the needed standards. We also tested out the functions' logic to make sure they work as intended.

- **Codebase comparisons** - After this was done, we took a look at the previous audits, audits of the NounsDao from which it is forked, compared the differences between the previous and new implementations. We tried to find any similar protocol types, compared their implementations and tried to find any general vulnerabilities.

## **4. Conclusions**

- The revolution protocol is like a community-run art gallery and marketplace where users submit art, and the community decides which piece is most deserving of being turned into a unique digital collectible and auctioned off. The proceeds benefit the artist and the platform, while the winner gets the exclusive artwork. Both the artwork and the artist's tokens give them voting power to influence future art selections in the community.

- The codebase is well-designed, a number of security measures had also been put in place, as a result we believe the team has done a good job overall. A bit of retouching is still needed to improve the overall protocol security and compliance to needed EIP standards.  

- We recommend implementing a supportsinterface function in the VerbsToken contract to make it more compliant with the ERC721 nft standard. In the same vein, checks needs to be put in place in the tokenUri function to ensure that data is not returned for invalid tokenids.

- Before locking the CultureIndex and the Descriptor, consider checking if they have been set. The current implementation does not check. The locking property, should be reconsidered, or at the very least, an unlock function should be implemented. The pausable property of the ERC20TokenEmitter should also be reconsidered as a main part of the auction process depends on it.

- Consider implementing chainId parameter for the NFTs and the signatures in cases of hard forks and to protect from replay attacks.

- Input validations should be improved in the constructors and initializers, important parameters should have min and max caps to prevent user-griefing by extreme values. Parameters and address changes should also be two steps.

- SafeCast should be implemented safely casted to prevent oveflows. Custom errors should be used in place of revert strings

- Recommend using solidity(and openzeppelin) versions that are a bit older just to be on a safer side. The latest ones might have some not yet discovered vulnerabilities lurking. Important to also note that the solidity version in use is not supported on the L2 chains, so contracts will not be deployable.

- As, there are a lot of parameters that can be changed by the owners at any time, a timelock should be introduced to give users time to react to these changes. It also gives time to roll back changes in case they break something. 

- Test coverage should be imporved from about 88%. It helps to catch basic bugs and improves code modularity.

- Above all, we recommend mitigting discovered issues, implementing constant upgrades and audits to keep the protocol always secure.



  
  
  




### Time spent:
48 hours