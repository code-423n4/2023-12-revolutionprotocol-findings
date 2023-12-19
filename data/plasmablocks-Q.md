## Issue 1

The Revolution protocol will be deployed on not just ethereum mainnet but also a few L2's (such as base and optimism). The current contracts use solidity version `0.8.22` which will not be deployable on any chain other than ethereum due to the introduction of the `PUSH0` opcode in solidity version `0.8.20` which is only supported on ethereum. 


Update the solidity version in all contracts to `0.8.19`.


## Issue 2

When validating the mediaType of an art piece the protocol hardcodes the maximum expected value of the `metadata.mediaType` in this require method:

`require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= 5, "Invalid media type");`

If the protocol ever adds or remove mediaTypes this value will need to be updated.


## Link to affected code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L160

## Recommendation

Update the mediaType validation to dynamically read the max value of the `MediaType` enum.

`require(uint8(metadata.mediaType) > 0 && uint8(metadata.mediaType) <= (type(MediaType).max), "Invalid media type");`