## \[G‑01\] Save gas by preventing zero amount in mint() 

```
function _mint(address _to, uint256 _amount) private {
        token.mint(_to, _amount);
    }
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L108

```
_mint(creatorsAddress, uint256(totalTokensForCreators));
```

&nbsp;https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L202

## \[G-02\] For `uint` use `!= 0` instead of `> 0`

```
require(msg.value > 0, "Must send ether");
```

&nbsp;https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L160

```
if (totalTokensForBuyers > 0) {
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L210

```
require(amount > 0, "Amount must be greater than 0");
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L238

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L255

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L272

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L363

## \[G-03\] Avoiding Initialization of the variables to it's default value

```
uint256 bpsSum = 0;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L205

```
uint256 ethPaidToCreators = 0;
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L380

## \[G-4\] Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.

```
string private _contractURIHash = "QmQzDwaZ7yQxHHs7sQQenJVB89riTSacSGcJRv9jtHPuz5";
```

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L63

## \[G-5\]Amounts should be checked for 0 before calling a transfer

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L419C5-L442C10