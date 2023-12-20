### Gas Optimisation Report

#### Low Issues:

**[L-1]:**
- **Location**: [CultureIndex.sol#L138](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L138)
- **Summary**: Add a check to ensure `_cultureIndexParams.minVoteWeight` is greater than 0. This validation is crucial for maintaining the integrity of the voting mechanism and avoiding potential issues with zero or negative weights.

**[L-2]:**
- **Location**: [TokenEmitterRewards.sol#L18](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18)
- **Summary**: The inspection `if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();` is redundant. The `computeTotalReward` function returns 2.5% of the passed value, which will always be less than `msgValue`. Suggestion is to remove this code to optimize gas usage.

#### Non-critical:

**[N-1]:**
- **Location**: [AuctionHouse.sol#L278](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L278)
- **Summary**: Bound the `timeBuffer` value to prevent extreme or invalid configurations that could affect auction timing and efficiency.

**[N-2]:**
- **Location**: [AuctionHouse.sol#L288](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L288)
- **Summary**: Bound the `reservePrice` value to ensure it remains within reasonable and manageable limits, enhancing the stability and predictability of the auction process.

**[N-3]:**
- **Location**: [AuctionHouse.sol#L298](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L298)
- **Summary**: Bound the `minBidIncrementPercentage` to prevent excessively high or low increments, which could lead to inefficient auction dynamics.

**[N-4]:**
- **Location**: [CultureIndex.sol#L134](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L134)
- **Summary**: Bound `_cultureIndexParams.erc721VotingTokenWeight` to ensure it stays within practical limits, maintaining a balanced and fair voting system.
