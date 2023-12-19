# Redundant check wasting gas

## Description

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L438

In the `_verifyVoteSignature()` function of the `CultureIndex.sol` there is a check whether address parameter `from` equals zero or not, but this `if` statement is redundant and wasting gas as the next `if` statement covers that check:

```solidity
        // Ensure to address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();
```

## Remediation

Remove redundant `if` statement:

```diff
        // Ensure to address is not 0
--      if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();
```