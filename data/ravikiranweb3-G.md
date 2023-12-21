1) CultureIndex::createPiece : The population of array and emitting events can be done in one loop

```` 
         for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

        // Emit an event for each creator
        for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }
````
2) AuctionHouse:_settleAuction(): Created array could be loaded into memory to save gas.
verbs.getArtPieceById(_auction.verbId).creators could be loaded into memory array for looping each creator.
````
  //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                if (creatorsShare > 0 && entropyRateBps > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
                      ....
                }
```