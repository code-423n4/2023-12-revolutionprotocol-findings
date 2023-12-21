## [GAS-1] Ensure ` paymentAmount `  is greater than zero before sending ETH to save further execution gas
In some situations  ` paymentAmount `   can be zero so there is a need to check if   ` paymentAmount `   and not try to send zero ETH afterwards in order to save execution gas.

``` //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);//@audit div before mul
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator//@audit gas: if amout is zero dont send
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);`


File: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L389C24-L394C86

## [GAS-2]  CultureIndex.sol#createPiece(...) function consumes much gas because of the multilpe reads and write from the contract's state.

There are multiple reads and writes to state in the createPiece function which will result to multiple costly SLOADS and SSTORE opcodes.

It will cost less gas if the ` newPiece `  is made a memory location variable before finally assigning it to state this way.
` ArtPiece storage newPiece;`
- The assign every property of ` newPiece `  before

` pieces[pieceId] = newPiece;`

The ` createPiece(...)`  function 

````` function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);

        // Validate the media type and associated data
        validateMediaType(metadata);

        uint256 pieceId = _currentPieceId++;

        /// @dev Insert the new piece into the max heap
        maxHeap.insert(pieceId, 0);

        ArtPiece storage newPiece = pieces[pieceId];

        newPiece.pieceId = pieceId;
        newPiece.totalVotesSupply = _calculateVoteWeight(
            erc20VotingToken.totalSupply(),
            erc721VotingToken.totalSupply()
        );
        newPiece.totalERC20Supply = erc20VotingToken.totalSupply();
        newPiece.metadata = metadata;
        newPiece.sponsor = msg.sender;
        newPiece.creationBlock = block.number;//@audit dont' do that on L2 bro
        newPiece.quorumVotes = (quorumVotesBPS * newPiece.totalVotesSupply) / 10_000;

        for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

        // Emit an event for each creator
        for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }

        return newPiece.pieceId;
    }`  

