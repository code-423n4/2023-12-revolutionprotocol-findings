# Typographical Error in Code Comments

## Summary

The AuctionHouse contract incorrectly refers to its ERC721 token by a different name.

&nbsp;

## Vulnerability Details

In [line 354](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L354C33-L354C33) of the AuctionHouse contract, the comments mistakenly mention the ERC721 token to burn as "Noun" instead of "Verb." The correct reference should be "Verb," as this contract was forked from the original Noun contract.

&nbsp;

## Impact

This typo can mislead and confuse reviewers of the contract.

&nbsp;

## Tools Used

Manual review

&nbsp;

## Recommended Mitigation Steps

Correct the typo by changing "Noun" to "Verb" in the comments.
