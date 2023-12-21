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


&nbsp;
&nbsp;
&nbsp;
# Lack of Event Emission After Sensitive Actions

## Summary

Best practices recommend emitting events after every state change to enhance transparency and off-chain tracking.

&nbsp;

## Vulnerability Details

The `AuctionHouse::initialized()` function lacks event emissions for the following state changes:

- `timeBuffer = _auctionParams.timeBuffer`
- `reservePrice = _auctionParams.reservePrice`
- `minBidIncrementPercentage = _auctionParams.minBidIncrementPercentage`
- `duration = _auctionParams.duration`
- `creatorRateBps = _auctionParams.creatorRateBps`
- `entropyRateBps = _auctionParams.entropyRateBps`

&nbsp;

## Impact

Poor Developer Experience: The absence of events or poorly designed events makes it challenging for blockchain applications and tools that monitor blockchain activities to keep track of the actions performed within the AuctionHouse contract.

&nbsp;

## Tools Used

Manual review

&nbsp;

## Recommended Mitigation Steps

It is strongly recommended to emit events after every state change and to index event parameters of type address to improve transparency and off-chain tracking. Consider adding events for each of the mentioned state changes in the `AuctionHouse::initialized()` function.

&nbsp;
&nbsp;
&nbsp;
# Initialization Timeframe Delay Causing Creators to Miss Rewards

## Summary

The `ERC20TokenEmitter` contract experiences an initialization timeframe delay, resulting in a period where creators receive zero rewards for all tokens bought.

&nbsp;

## Vulnerability Details

The variables `creatorRateBps` and `setEntropyRateBps` are not set during contract initialization, implying their values default to zero. This leads to zero tax being paid to creators (`creatorDirectPayment` and `totalTokensForCreators`) when users buy tokens. If this is intended, it should be clearly documented in the project's documentation. Otherwise, the current implementation deprives creators of their rewards.

&nbsp;

## Impact

Creators may feel frustrated for missing out on potential rewards, particularly if the amounts are significant.

&nbsp;

## Tools Used

Manual review

&nbsp;

## Recommended Mitigation Steps

Set the `creatorRateBps` and `setEntropyRateBps` variables in the `initialize()` function if the protocol intends creators to benefit from token buys from the start. Alternatively, clearly document these mechanics in the project's documentation if the delay in rewarding creators is intentional.


# Misleading Comment in MaxHeap Contract

## Summary

The comment above MaxHeap::insert() says "/// @dev The function will revert if the heap is full," but the heap doesn't have a max capacity.

## Vulnerability Details

The MaxHeap contract does not explicitly define a maximum size for the heap, nor does it enforce a maximum size through the code. In Solidity, mappings, such as the heap mapping used to represent the heap data structure, do not have a predefined size limit and can grow dynamically as long as there is enough gas to perform the operations and the Ethereum block gas limit is not exceeded.

## Impact

This inaccuracy can mislead and confuse reviewers of the contract.

## Tools Used

Manual review

## Recommended Mitigation Steps

Remove the comment.


&nbsp;
&nbsp;
&nbsp;
# Convert Non-State Accessing Functions to Pure to Save Gas

## Summary

The following functions can be restricted to pure functions: `transfer(); _transfer(); transferFrom; approve; _approve; _approve; _spendAllowance`.

## Vulnerability Details

Due to the non-transferable nature of the token, functions related to transfer, such as `_transfer(); transferFrom; approve; _approve; _approve; _spendAllowance`, do not read or write to the contract storage. Consequently, marking these functions as pure or view can enhance gas efficiency.

## Impact

Saving gas costs for users.

## Tools Used

Manual review

## Recommended Mitigation Steps

Change the following functions to pure functions: `_transfer(); transferFrom; approve; _approve; _approve; _spendAllowance`.


 
&nbsp; 
&nbsp;  
&nbsp;  
# Misleading Comment in MaxHeap Contract

## Summary

In [line 16](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/NontransferableERC20Votes.sol#L16) of `NontransferableERC20Votes`, the comment implies the token can be transferred, which is not the case for the protocol's token.

## Vulnerability Details

The `NontransferableERC20Votes` contract does not allow the transfer of tokens from one account to another. However, the comment describes the token as having such properties.

## Impact

This inaccuracy can mislead and confuse reviewers of the contract.

## Tools Used

Manual review

## Recommended Mitigation Steps

Correct the error in the comment. Clearly state that the token is non-transferable and update the comment to accurately reflect the token's capabilities.
