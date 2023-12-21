# Non-Critical Findings

## \[N-01\] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-02\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## \[N‑03\] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10\*\*18)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.

## \[N-04\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

## \[N-05\] Use constants for numbers

==In several locations in the code, numbers like 1e36, 100e18, 1e27 are used...==  
https://github.com/code-423n4/2021-05-yield-findings/issues/3#issuecomment-852039791

## \[N-06\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-07\] Use the delete keyword instead of assigning a value of 0

Using the ‘delete’ keyword instead of assigning a ‘0’ value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use delete instead 0 value assign , it will be gas saved.

## \[N-08\] Function writing that does not comply with the Solidity Style Guide

Context  
All Contracts

Description  
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor  
receive function (if exists)  
fallback function (if exists)  
external  
public  
internal  
private  
within a grouping, place the view and pure functions last