The Revolution protocol will be deployed on not just ethereum mainnet but also a few L2's (such as base and optimism). The current contracts use solidity version `0.8.22` which will not be deployable on any chain other than ethereum due to the introduction of the `PUSH0` opcode in solidity version `0.8.20` which is only supported on ethereum. 


Update the solidity version in all contracts to `0.8.19`.