# [N1] Repeated for loop

In the CultureIndex#createPiece function, there is a repeated for loop:

```solidity
    function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);
        ....
         for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
        }
        ....
        for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }
        ....
    }
```
It is recommended to merge the two for loops.


# [N2] The price of a token can be calculated in a variety of ways

The `ERC20TokenEmitter#bugToken` function ultimately uses `vrgdac.yToX` to obtain the number of tokens that can be purchased based on the number of incoming eth payments.

There are several ways to calculate the price of a token:
In the `bugToken` function, `getTokenQuoteForEther`(->vrgdac.yToX), the `yToX` parameter `sold` has an effect on the number of tokens returned (the price of the token),

The passed argument to `yToX` `sold` is stored in the `emittedTokenWad` variable of the `ERC20TokenEmitter` contract. The `bugToken` function increases the value of `emittedTokenWad` after calling `getTokenQuoteForEther` twice, so: 
1. If `emittedTokenWad` is added once for each call, the number of tokens returned is different from that of `emittedTokenWad` after two calls.

bugToken first divides the eth paid by the user into two parts, and then calculates the number of tokens that can be obtained for each part ,so:
2. If the number of tokens that can be obtained is calculated first, and then the obtained tokens are allocated in proportion, the results are also different.

The problem is that users want to know the number of tokens they can get before buying tokens, so users can call other functions in `ERC20TokenEmitter` to calculate, but due to the token allocation strategy in the protocol and the `vrgdac`. Users cannot accurately know the number of tokens that can be obtained, and there are many different calculation methods, and the actual purchase of tokens will not be consistent with the expected.

Therefore, it is recommended to use the same token calculation method in `ERC20TokenEmitter` contract and return the calculation result through a `view function`.

```solidity
    function getTokenQuoteForEther(uint256 etherAmount) public view returns (int gainedX) {
        require(etherAmount > 0, "Ether amount must be greater than 0");
        return
            vrgdac.yToX({
                timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
                sold: emittedTokenWad,
                amount: int(etherAmount)
            });
    }
```