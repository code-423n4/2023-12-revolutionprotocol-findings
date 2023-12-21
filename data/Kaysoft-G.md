## [GAS-1] Ensure ` paymentAmount `  is greater than zero before sending ETH to save further execution gas
In some situations  ` paymentAmount `   can be zero so there is a need to check if   ` paymentAmount `   and not try to send zero ETH afterwards in order to save execution gas.

``` //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);//@audit div before mul
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator//@audit gas: if amout is zero dont send
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);`


File: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L389C24-L394C86

