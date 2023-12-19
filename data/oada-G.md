## [G-01]Remove initializer modifier
Remove the initializer modifier and call the `initialize()` function within the constructor to reduce gas consumption.
```solidity
packages\revolution\src\CultureIndex.sol

  116:    ) external initializer { 
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L116
## [G-02]Remove initializer modifier
The constructor fails to check the zero address. If a zero address is passed in by mistake, the contract may be deployed again, consuming a large amount of gas.
```solidity
packages\revolution\src\CultureIndex.sol

  92:  constructor(address _manager) payable initializer {
  93:      manager = IRevolutionBuilder(_manager);
  94:  }

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L92-L94