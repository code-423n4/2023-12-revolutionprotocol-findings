1.Potentially unsafe usage of `ecrecover` return value
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L435
The ecrecover function may return a non-zero value even if the signature is invalid. It's unsafe to use the return value of ecrecover for any purpose other than comparing it with a valid signer value.

2.verbId == _currentVerbId is not a valid piece ID
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L274
```solidity
-require(verbId <= _currentVerbId, "Invalid piece ID");
+require(verbId < _currentVerbId, "Invalid piece ID");
```