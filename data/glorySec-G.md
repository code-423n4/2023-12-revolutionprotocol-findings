1. No Need to check *recoveredAddress* is not equal to *address(0)*, if from is checked! 
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L441

This can be safely changed 
```if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();```
To this
`if (recoveredAddress != from) revert INVALID_SIGNATURE();`