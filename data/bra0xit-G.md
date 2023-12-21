### Two for-loops can be combined into one (Saves 72 Gas per piece created)

The two for-loops are iterating over the same array and could thus be combined into one to save gas on execution, saving gas on deployment as well as simplifying the codebase.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L236

### Two for-loops can be combined into one (Saves 27 846 gas for a multi vote with 2 addresses)

The two for-loops are iterating over the same array and could thus be combined into one to save gas on execution, saving gas on deployment as well as simplifying the codebase.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L403

### Two for-loops can be combined into one (Saves gas for each deposit batch)

The two for-loops are iterating over the same array and could thus be combined into one to save gas on execution, saving gas on deployment as well as simplifying the codebase.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/RevolutionProtocolRewards.sol#L62