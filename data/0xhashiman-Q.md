#Summary
Some functions within the protocol operate independently without invoking other functions, which means its impossible to reenter the state.Despite their standalone nature, these functions are safeguarded by a 'nonReentrant' modifier.


#Vulnerability Detail
If a method doesn't involve any external calls, reentrancy is impossible, allowing for the omission of this modifier in such methods.


#Code Snippet
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L209-L214

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L230-L236

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L252-L256

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L309-L313


#Tool used

Manual Review

#Recommendation
Remove nonReentrant modifier on function that don't have any external calls


