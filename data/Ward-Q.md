## [L-01] Possible to set the `timeBuffer` too big that it freezes the bidder's funds.

It's possible to set timeBuffer to a big value with [setTimeBuffer()](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L277-L281) function, since there is no control.

```solidity
File: AuctionHouse.sol
function setTimeBuffer(uint256 _timeBuffer) external override onlyOwner {
        timeBuffer = _timeBuffer;

        emit AuctionTimeBufferUpdated(_timeBuffer);
    }
```
**Solution: Consider an upper limit for timeBuffer.**