L-1 `_settleAuction` will fail cuz owner is not payable 
in the function _safeTransferETHWithFallback it pass owner() as address while it need that address to be payable the function will fail

_safeTransferETHWithFallback(owner(), auctioneerPayment);

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L419


N-1 wrong comment 

comment appears to be misleading, as it mentions a "struct" while the code actually defines a mapping.  

/// @notice Struct to represent an item in the heap by it's ID
mapping(uint256 => uint256) public heap;

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L64

N-2 auction.bidder is already payable 

auction.bidder = payable(bidder); 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L188