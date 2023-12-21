### Low. Replace gasLeft based logic in AuctionHouse.sol.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L311

Instead of checking the gas needed, which can change over time, check if the conditions for minting a verbtoken are met. If the conditions are met, call verbs.mint() without catching reverts. If the conditions are not met, _pause() the smart contract. Conditions for minting:
```solidity
pieceId, votes = maxHeap.getMax() .heap(0)) 
if (MaxHeap.size() > 0 && votes >= cultureIndex.pieces(pieceId).quorumVotes()) {
    // mint
} else {
    _pause();
}
``` 