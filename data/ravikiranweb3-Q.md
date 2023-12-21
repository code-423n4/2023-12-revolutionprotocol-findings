1) VerbsToken::_mintTo(): 
    creationBlock is not assigned for newPiece which could be the block in which the token was actually minted. 

2) AuctionHouse::createBid():
    auction state variable is not initially assigned in the constructor. The contract is put on pause() and on unpause(), a new auction is started if valid to start. So, it is recommended that createBid is also valid when not paused. 

Recommendation is to assign whenNotPaused() modifier to createBid() so that invalid auction state variable can be prevented for first run.