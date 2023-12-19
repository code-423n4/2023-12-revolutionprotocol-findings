## [G-01]Remove initializer modifier
Remove the initializer modifier and call the `initialize()` function within the constructor to reduce gas consumption.
```java
packages\revolution\src\CultureIndex.sol

  116:    ) external initializer { 
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L116