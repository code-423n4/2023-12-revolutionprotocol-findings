### Redundant Use of `initializer` Modifier in Constructor
###### Decription:

The constructor of the smart contract defines an ``initializer modifier, but it is unnecessary since the `initializer` function can only be triggered by the `manager`. The constructor itself takes the `manager` as a parameter, making the `initializer` modifier redundant.

###### Recommendation:
Remove the `initializer` modifier from the constructor, as it serves no functional purpose in this context.

###### Lines of Github:
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L92https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L44https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L68https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L95https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L116

