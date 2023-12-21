[L-01] `Set MaxDuration for auctions so owners cant create Auctions that could last forever.` 
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L139

[L-02] `The zero address check for creator address is redundant.` 
In ERC20TokenEmitter, The zero address check for creator address is done mutiple times all over the code base. 
**Recomendation**
Move the zero address check for the creator address to the initialize function and remove the check in the if statement in line 201, since the zero address check is done on the initialize function, you can safely use the state variable any where in the code and when its being changed, the function `setCreatorAdress()` also ensures the new address its not a zero address. 

[L-03] `BlockNumbers differ across layer1 and layer2. `
Layer 2 solutions periodically commit their state to the Ethereum mainnet, their block numbers may not directly correlate with the mainnet's block numbers. The block numbers on a Layer 2 chain are specific to its chain and do not affect or reflect the block numbers on the Ethereum mainnet. Hence its important to note that using block.number for creation block may differ across platforms
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L233C47-L233C47

[L-04] `A bid placed on an Artpiece is permanent and cannot be cancelled`
**Recommendation**
Consider adding Functionality for bidders to cancel their bids. With the current implementation any bid placed on an art piece is permanent and cannot be cancelled unless someone places a higher bid. 