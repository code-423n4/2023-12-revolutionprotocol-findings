# [I-1] `NontrasferableERC20Votes` should inherit `INontrasferableERC20Votes`

## Description

Contract implementations should inherit their interfaces. Extending an interface ensures that all function signatures are correct, and catches mistakes introduced (e.g. through errant keystrokes)

## Proof of Concept

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/NontransferableERC20Votes.sol#L29
```solidity
contract NontransferableERC20Votes is Initializable, ERC20VotesUpgradeable, Ownable2StepUpgradeable {
```
Does not Inherit `INontrasferableERC20Votes` defined [here](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/interfaces/INontransferableERC20Votes.sol)

## Recommended Mitigation

```diff
+ import { INontrasferableERC20Votes } from "./interfaces/IRevolutionBuilder.sol";
...
- contract NontransferableERC20Votes is Initializable, ERC20VotesUpgradeable, Ownable2StepUpgradeable {
+ contract NontransferableERC20Votes is Initializable, ERC20VotesUpgradeable, Ownable2StepUpgradeable, INontrasferableERC20Votes {
```