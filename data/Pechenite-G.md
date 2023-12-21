## [G-01] Disallow setting creator BPS to 0

In the `CultureIndex` contract, the `validateCreatorsArray()` function doesn't check if the creator BPS is 0. Consider adding a check to ensure that the creator BPS is not 0. This would skip the following logic in `AuctionHouse.sol#_settleAuction()`, when creator BPS is 0, which would save on gas:

```solidity
384:                     for (uint256 i = 0; i < numCreators; i++) {
385:                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
386:                        vrgdaReceivers[i] = creator.creator;
387:                        vrgdaSplits[i] = creator.bps;
388:                        if (creator.bps != 0) { 
389:
390:                            //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
391:                            uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
392:                            ethPaidToCreators += paymentAmount;
393:
394:                            //Transfer creator's share to the creator
395:                            _safeTransferETHWithFallback(creator.creator, paymentAmount);
396:                        }
397:                    }
```
[Link](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#384)


To prove that let's run the following test inside `AuctionSettling.t.sol`:

```solidity
    function testSettlingAuctionWithMultipleCreatorsZeroBPS() public {
        uint nCreators = 100;

        uint256 creatorRate = (auction.creatorRateBps());
        uint256 entropyRate = (auction.entropyRateBps());

        address[] memory creatorAddresses = new address[](nCreators);
        uint256[] memory creatorBps = new uint256[](nCreators);
        uint256 totalBps = 0;

        // Assume n creators with equal share
        for (uint256 i = 0; i < nCreators; i++) {
            creatorAddresses[i] = address(uint160(i + 1)); // Example creator addresses
            if (i == nCreators - 1) {
                creatorBps[i] = 10_000;
            } else {
                creatorBps[i] = 0;
            }

            totalBps += creatorBps[i];
        }

        uint256 verbId = createArtPieceMultiCreator(
            "Multi Creator Art",
            "An art piece with multiple creators",
            ICultureIndex.MediaType.IMAGE,
            "ipfs://multi-creator-art",
            "",
            "",
            creatorAddresses,
            creatorBps
        );

        auction.unpause();

        uint256 bidAmount = auction.reservePrice();
        vm.deal(address(21_000), bidAmount + 1 ether);
        vm.startPrank(address(21_000));
        auction.createBid{ value: bidAmount }(verbId, address(21_000));
        vm.stopPrank();

        vm.warp(block.timestamp + auction.duration() + 1); // Fast forward time to end the auction

        // Track balances before auction settlement
        uint256[] memory balancesBefore = new uint256[](creatorAddresses.length);
        uint256[] memory mockWETHBalancesBefore = new uint256[](creatorAddresses.length);
        uint256[] memory governanceTokenBalancesBefore = new uint256[](creatorAddresses.length);
        for (uint256 i = 0; i < creatorAddresses.length; i++) {
            balancesBefore[i] = address(creatorAddresses[i]).balance;
            governanceTokenBalancesBefore[i] = erc20Token.balanceOf(creatorAddresses[i]);
            mockWETHBalancesBefore[i] = MockWETH(payable(weth)).balanceOf(creatorAddresses[i]);
        }

        // Track expected governance token payout
        uint256 etherToSpendOnGovernanceTotal = uint256(
            (bidAmount * creatorRate) / 10_000 - (bidAmount * (entropyRate * creatorRate)) / 10_000 / 10_000
        );

        uint256 expectedGovernanceTokenPayout = uint256(
            erc20TokenEmitter.getTokenQuoteForEther(
                etherToSpendOnGovernanceTotal - erc20TokenEmitter.computeTotalReward(etherToSpendOnGovernanceTotal)
            )
        );

        auction.settleCurrentAndCreateNewAuction();


        //assert auctionHouse balance is 0
        assertEq(address(auction).balance, 0);

        // Verify each creator's payout
        for (uint256 i = 0; i < creatorAddresses.length; i++) {
            uint256 expectedEtherShare = uint256(((bidAmount) * creatorBps[i] * creatorRate) / 10_000 / 10_000);

            //either the creator gets ETH or WETH
            assertEq(
                address(creatorAddresses[i]).balance - balancesBefore[i] > 0
                    ? address(creatorAddresses[i]).balance - balancesBefore[i]
                    : MockWETH(payable(weth)).balanceOf(creatorAddresses[i]) - mockWETHBalancesBefore[i],
                (expectedEtherShare * entropyRate) / 10_000,
                "Incorrect ETH payout for creator"
            );

            assertApproxEqAbs(
                erc20Token.balanceOf(creatorAddresses[i]) - governanceTokenBalancesBefore[i],
                uint256((expectedGovernanceTokenPayout * creatorBps[i]) / 10_000),
                // "Incorrect governance token payout for creator",
                1
            );
        }

        // Verify ownership of the verb
        assertEq(erc721Token.ownerOf(verbId), address(21_000), "Verb should be transferred to the highest bidder");
        // Verify voting weight on culture index is 721 vote weight for winning bidder
        assertEq(
            cultureIndex.getVotes(address(21_000)),
            cultureIndex.erc721VotingTokenWeight() * 1e18,
            "Highest bidder should have 10 votes"
        );
    }
```

Before: 37,296,585
After: 37,061,470

Difference: 235,115