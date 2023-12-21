# QA Findings

## First QA Finding
### A user can ensure winning the auction if he has an offchain automation.

A user can ensure to win the auction(Spending more ETH & Gas than others) if he has an automation that Trigger a call offchain due to the change in the balance of WETH
#### Code Involved

    function _safeTransferETHWithFallback(address _to, uint256 _amount) private {
        // Ensure the contract has enough ETH to transfer
        if (address(this).balance < _amount) revert("Insufficient balance");

        // Used to store if the transfer succeeded
        bool success;

        assembly {
            // Transfer ETH to the recipient
            // Limit the call to 50,000 gas
            success := call(50000, _to, _amount, 0, 0, 0, 0)
        }

        // If the transfer failed:
        if (!success) {
            // Wrap as WETH
            IWETH(WETH).deposit{ value: _amount }();

            // Transfer WETH instead
            bool wethSuccess = IWETH(WETH).transfer(_to, _amount);

            // Ensure successful transfer
            if (!wethSuccess) revert("WETH transfer failed");
        }
    }
## Second QA Finding
### MinBidIncrementPercentage should be more than 0, otherwise, it could lead to an infinite duration auction

MinBidIncrementPercentage should always be more than 0. If that is not stipulated in any place in the code, you could set it to be equal. This way, a malicious user could spend some gas every time the endTime is close to being reached. If that happens, the auction will continue forever without actual bids
#### Code Involved
    function createBid(uint256 verbId, address bidder) external payable override nonReentrant {
        IAuctionHouse.Auction memory _auction = auction;

        //require bidder is valid address
        require(bidder != address(0), "Bidder cannot be zero address");
        require(_auction.verbId == verbId, "Verb not up for auction");
        //slither-disable-next-line timestamp
        require(block.timestamp < _auction.endTime, "Auction expired");
        require(msg.value >= reservePrice, "Must send at least reservePrice");
        require(
            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),
            "Must send more than last bid by minBidIncrementPercentage amount"
        );
