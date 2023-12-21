1) VerbsToken::_mintTo(): 
    creationBlock is not assigned for newPiece which could be the block in which the token was actually minted. 

2) AuctionHouse::createBid():
    auction state variable is not initially assigned in the constructor. The contract is put on pause() and on unpause(), a new auction is started if valid to start. So, it is recommended that createBid is also valid when not paused. 

Recommendation is to assign whenNotPaused() modifier to createBid() so that invalid auction state variable can be prevented for first run.

3) AuctionHouse::duration state variable:
    The auction house does not have the ability to adjust the duration of auction at a later point, If there was a need to adjust the duration, it is better to add an admin function to manage the auction time window for future auction only. The active auction time window should no be updated.
