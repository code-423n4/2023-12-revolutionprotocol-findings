1. In ```ERC20TokenEmitter::setCreatorsAddress```, the ```owner``` can update the ```creatorsAddress```. However, there's no need to use the ```nonReentrant``` guard here. Suggest to remove it.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L309


2. In ```CultureIndex::createPiece()```, it uses the first for-loop to store the creators and another for-loop to emit event for each creator. To save gas, can combine them into one for-loop.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L236-L245


        for (uint i; i < creatorArrayLength; i++) {
            newPiece.creators.push(creatorArray[i]);
        }

        emit PieceCreated(pieceId, msg.sender, metadata, newPiece.quorumVotes, newPiece.totalVotesSupply);

        // Emit an event for each creator
        for (uint i; i < creatorArrayLength; i++) {
            emit PieceCreatorAdded(pieceId, creatorArray[i].creator, msg.sender, creatorArray[i].bps);
        }