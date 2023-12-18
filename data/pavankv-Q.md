
## 1. Add require to check zeroness of the given Array :-

The Internal function called `validateCreatorsArray()` which do not have mechanism to check zero length of the `CreatorBps[] calldata creatorArray` given argument which can pass the first require statement `require(creatorArrayLength <= MAX_NUM_CREATORS,)` in the function. So if we add proper validation mechanism with proper revert statement makes system more robustness

**Before**
```solidity

    function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
        uint256 creatorArrayLength = creatorArray.length; // Zero ness check of array QA and Analysis.
        //Require that creatorArray is not more than MAX_NUM_CREATORS to prevent gas limit issues
        require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

```

**After**
```solidity
    function validateCreatorsArray(CreatorBps[] calldata creatorArray) internal pure returns (uint256) {
        uint256 creatorArrayLength = creatorArray.length; // Zero ness check of array QA and Analysis.
        require(creatorArrayLength != 0 ,"ER");
        //Require that creatorArray is not more than MAX_NUM_CREATORS to prevent gas limit issues
        require(creatorArrayLength <= MAX_NUM_CREATORS, "Creator array must not be > MAX_NUM_CREATORS");

```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L179C1-L182C105

## 2. Improper implementation of require statement.
If we look into the below comment in the code state that `minimum vote weight required in order to vote` means with minimum vote weight can be voted. 

```solidity
   /// @notice The minimum vote weight required in order to vote
    uint256 public minVoteWeight; // QA and analysis
```
But use cases in the of `minVoteWeight` variable is look like even having minimum vote weight cannot be voted.

```solidity
    function _vote(uint256 pieceId, address voter) internal {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        require(voter != address(0), "Invalid voter address"); // Gas no use
        require(!pieces[pieceId].isDropped, "Piece has already been dropped");
        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");

        uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);
        require(weight > minVoteWeight, "Weight must be greater than minVoteWeight"); // @audit check here

```

Recommend:-
```soldiity
        uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);
        require(weight >= minVoteWeight, "Weight must be greater than minVoteWeight"); // @audit check here

```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L312C1-L314C86
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L50C1-L51C34

## 3. User cannot get the current pieceId and current piece id vote :-
We can look the below functions where it allows only below the `currentPieceID` variable but it will be increamented on the `createPiece()` function. Our recommedation allows user to get the current piece ID and it's vote value.
```solidity
    /**
     * @notice Fetch an art piece by its ID.
     * @param pieceId The ID of the art piece.
     * @return The ArtPiece struct associated with the given ID.
     */
    function getPieceById(uint256 pieceId) public view returns (ArtPiece memory) {
        require(pieceId < _currentPieceId, "Invalid piece ID"); // QA @audit check here
        return pieces[pieceId];
    }

    /**
     * @notice Fetch the list of votes for a given art piece.
     * @param pieceId The ID of the art piece.
     * @return An array of Vote structs for the given art piece ID.
     */
    function getVote(uint256 pieceId, address voter) public view returns (Vote memory) {
        require(pieceId < _currentPieceId, "Invalid piece ID"); // QA @audit check here
        return votes[pieceId][voter];
    }
```

recommendation
```solidity
        require(pieceId <= _currentPieceId, "Invalid piece ID"); // QA @audit check here
        
        require(pieceId <= _currentPieceId, "Invalid piece ID"); // QA @audit check here

```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L451C1-L464C6

## 4. Add check that is Piece is available to drop or not :-
n the `dropTopVotedPiece()` function, which is publicly accessible, any code can call it regardless of its location. However, this function lacks validation for a previously dropped piece. This means an already dropped piece can be dropped again, ultimately setting the isDropped variable back to `true` and overwriting its previous state.

```solidity
    function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");

        ICultureIndex.ArtPiece memory piece = getTopVotedPiece();
        require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");

        //set the piece as dropped
        pieces[piece.pieceId].isDropped = true;

```

Recommedation
```solidity
    function dropTopVotedPiece() public nonReentrant returns (ArtPiece memory) {
        require(msg.sender == dropperAdmin, "Only dropper can drop pieces");
	require(!pieces[piece.pieceId].isDropped , "ERROR");  // Qa @audit check here.
        ICultureIndex.ArtPiece memory piece = getTopVotedPiece();
        require(totalVoteWeights[piece.pieceId] >= piece.quorumVotes, "Does not meet quorum votes to be dropped.");

        //set the piece as dropped
        pieces[piece.pieceId].isDropped = true;

```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L519C1-L526C48

## 5. Wrong implementation of setter functions :-

The `setMinCreatorRateBps()` function is intended to set the minimum creator rate in bps. However, there is an condition in the require statement. It incorrectly requires that the last `miniCreatorRateBps` value must be less than the argument `_miniCreatorRateBps`. This means that the `miniCreatorRateBps` variable can only continue to increase, even if a situation arises where it needs to be decreased. Even an admin would be unable to lower the minimum creator rate. 
```solidity
        //ensure new min rate cannot be lower than previous min rate
        require(
            _minCreatorRateBps > minCreatorRateBps, // != QA
            "Min creator rate must be greater than previous minCreatorRateBps"
        ); //Analysis and QA.

```

Recommendation:-
```solidity
        //ensure new min rate cannot be lower than previous min rate
        require(
            _minCreatorRateBps != minCreatorRateBps, // != QA @audit check here.
            "Min creator rate must be greater than previous minCreatorRateBps"
        ); //Analysis and QA.

```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L233C1-L246C6

Scenarion -2 
Similar to the `setMinCreatorRateBps()` function, the `setCreatorRateBps()` function also suffers from an erroneous condition in its require statement. It necessitates that the provided `_creatorRateBps` value be greater than or equal to the last recorded miniCreatorRateBps value. This restriction essentially renders setting the rate to the existing value pointless and effectively restricts the function to solely increasing the minimum creator rate. Consequently, even administrators are denied the ability to decrease the rate. We recommend implementing a proper mechanism within this setter function to facilitate controlled adjustments in both directions.

```solidity
    function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
        require(
            _creatorRateBps >= minCreatorRateBps, // QA
            "Creator rate must be greater than or equal to minCreatorRateBps"
        );
        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
        creatorRateBps = _creatorRateBps;

        emit CreatorRateBpsUpdated(_creatorRateBps);
    }

```

Recommendation
```solidity
    function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {
        require(
            _creatorRateBps != minCreatorRateBps, // QA @audit check here.
            "Creator rate must be greater than or equal to minCreatorRateBps"
        );
        require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
        creatorRateBps = _creatorRateBps;

        emit CreatorRateBpsUpdated(_creatorRateBps);
    }

```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L233C1-L246C6

## 6. Add a check for variable which will be rounding down to Zero.

In function `buyToken()` calls `_handleRewardsAndGetValueToSend()` function with four arguments and this function calls three functions `computeTotalReward()` , `computePurchaseRewards()` and `_depositPurchaseRewards()`. First we will look into the `computeTotalReward()` function
```solidity
	// 2.5% total
    uint256 internal constant DEPLOYER_REWARD_BPS = 25;
    uint256 internal constant REVOLUTION_REWARD_BPS = 75;
    uint256 internal constant BUILDER_REWARD_BPS = 100;
    uint256 internal constant PURCHASE_REFERRAL_BPS = 50;

    uint256 public constant minPurchaseAmount = 0.0000001 ether;
    uint256 public constant maxPurchaseAmount = 50_000 ether;	
	
	
    function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

        return
            (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000 + (paymentAmountWei * PURCHASE_REFERRAL_BPS) /10_000
             +
            (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000 + (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000;
    }
```

The `buyToken()` function seems to have a potential issue related to rounding. When `paymentAmountWie` exceeds the minimum threshold of 0.0000001 ether, it's multiplied by various bps and divided by `10_000`. However, this calculation consistently rounds down to zero for `msgValueRemaining`. This, in turn, affects downstream values like toPayTreasury and potentially other variables. Notably, there's no explicit check to ensure msgValueRemaining isn't inadvertently set to zero.

Recommendation:-
1. Implement a validation check for the return value. If it's equal to zero, add a variable to contribute a minimum amount to the system or treasury.
```solidity
    function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        //prevent treasury from paying itself
        require(msg.sender != treasury && msg.sender != creatorsAddress, "Funds recipient cannot buy tokens");

        require(msg.value > 0, "Must send ether");
        // ensure the same number of addresses and bps
        require(addresses.length == basisPointSplits.length, "Parallel arrays required");

        // Get value left after protocol rewards
        uint256 msgValueRemaining = _handleRewardsAndGetValueToSend(  // QA add zero check explain properly
            msg.value,
            protocolRewardsRecipients.builder,
            protocolRewardsRecipients.purchaseReferral,
            protocolRewardsRecipients.deployer
        );
        
        if(msgValueRemaining == 0 ) msgValueRemaining = MINIMUM_AMOUNT; //@audit check here.
        ....

```

2. If `msgValueRemaining` is equal to zero add different mechanism to calculate `toPayTreasury` , `creatorDirectPayment` and subsequent variables.

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L165C1-L170C11
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40C1-L52C6\

## 7. Computation can be done only if variable is greater than zero:-

As we mentioned in the above finding if `toPayTreasury` is zero due to rounding down, we shouldn't use it to calculate `totalTokensForBuyers` and `emittedTokenWad`.To avoid unnecessary calculations, we can add a check to see if `totalTokensForBuyers` is greater than zero before using it in the computation.

```solidity
        // Tokens to emit to buyers
        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0); //@audit check here

        //Transfer ETH to treasury and update emitted
        emittedTokenWad += totalTokensForBuyers;
```

Recommendation
```solidity
        // Tokens to emit to buyers
        int totalTokensForBuyers = toPayTreasury > 0 ? getTokenQuoteForEther(toPayTreasury) : int(0);

        //Transfer ETH to treasury and update emitted
        if(totalTokensForBuyers > 0) emittedTokenWad += totalTokensForBuyers;//@audit check here

```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L184C1-L187C49

## 8. Both for-loop can be combine :-
In the `batchVoteForManyWithSig()` function, two for-loops are used to verify signatures and vote for many pieces. These can be combined into a single loop, thereby reducing code size and makes code more concise.

```solidity
        for (uint256 i; i < len; i++) {
            if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();
        }

        for (uint256 i; i < len; i++) {
            _voteForMany(pieceIds[i], from[i]);
        }

```

Recommendation
```solidity
for (uint256 i; i < len; i++) {
    if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) {
        revert INVALID_SIGNATURE();
    }
    _voteForMany(pieceIds[i], from[i]);
}

```
Our recommendation doesn't cause any error can be implemented safely.

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L403C1-L409C10