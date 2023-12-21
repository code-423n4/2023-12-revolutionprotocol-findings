# Findings Summary

| |Issue|Instances|
|-|:-|:-:|
| [[G-0](#gas-0)] | Complex check will cuase more gas | 1 |
| [[G-1](#gas-1)] | Address zero check should be done in the beginning of the function in `CultureIndex::_verifyVoteSignature()` | 1 |
| [[G-2](#gas-2)] |Use `string.concat()` instead of `abi.encodePacked` to save gas | 1 |

---

[G-0] Complex check will cuase more gas <a id="gas-0"></gas>

In `CultureIndex::_vote()` function a check is made unnecessarily complex that will cause more gas. It could've been simpler.

```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/CultureIndex.sol

311        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");

```

Github: [311](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L311)

---

[G-1] Address zero check should be done in the beginning of the function in `CultureIndex::_verifyVoteSignature()` <a id="gas-1"></a>

In `CultureIndex::_verifyVoteSignature()`, `from` is checked after checking the signature. Add the check in the beginning to save gas.

```solidity

File: 2023-12-revolutionprotocol/packages/revolution/src/CultureIndex.sol

438        if (from == address(0)) revert ADDRESS_ZERO();

```

Github: [438](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L438)

---

[G-2] Use `string.concat()` instead of `abi.encodePacked` to save gas <a id="gas-2"></a>

In `VerbsToken::contractURI()`, `abi.encodePacked` is used to concat the two string. Use `string.concat()` instead that is more gas efficient.


```solidity
File: 2023-12-revolutionprotocol/packages/revolution/src/VerbsToken.sol

61    function contractURI() public view returns (string memory) {
62        return string(abi.encodePacked("ipfs://", _contractURIHash));
63    }

```

Github: [61-63](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L161C1-L163C6)