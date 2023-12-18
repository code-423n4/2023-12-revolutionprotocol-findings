## 1. Refactor the try statement :-

In the provided try statement, the _artPiece variable is declared and assigned within the block, but it is unused throughout the function's lifetime. Additionally, the assignment to _artPiece is unnecessary as the value is already assigned to artPiece.

```solidity
 try cultureIndex.dropTopVotedPiece() returns (ICultureIndex.ArtPiece memory _artPiece) { //Gas and QA
            artPiece = _artPiece;
            uint256 verbId = _currentVerbId++;

            ICultureIndex.ArtPiece storage newPiece = artPieces[verbId];

            newPiece.pieceId = artPiece.pieceId;
            newPiece.metadata = artPiece.metadata;
            newPiece.isDropped = artPiece.isDropped;
            newPiece.sponsor = artPiece.sponsor;
            newPiece.totalERC20Supply = artPiece.totalERC20Supply;
            newPiece.quorumVotes = artPiece.quorumVotes;
            newPiece.totalVotesSupply = artPiece.totalVotesSupply;

            for (uint i = 0; i < artPiece.creators.length; i++) {
                newPiece.creators.push(artPiece.creators[i]);
            }

            _mint(to, verbId);

            emit VerbCreated(verbId, artPiece);

            return verbId;
        }
```
Recommendation:-
```solidity
try cultureIndex.dropTopVotedPiece() { //Gas @audit check here.

            uint256 verbId = _currentVerbId++;

            ICultureIndex.ArtPiece storage newPiece = artPieces[verbId];

            newPiece.pieceId = artPiece.pieceId;
            newPiece.metadata = artPiece.metadata;
            newPiece.isDropped = artPiece.isDropped;
            newPiece.sponsor = artPiece.sponsor;
            newPiece.totalERC20Supply = artPiece.totalERC20Supply;
            newPiece.quorumVotes = artPiece.quorumVotes;
            newPiece.totalVotesSupply = artPiece.totalVotesSupply;

            for (uint i = 0; i < artPiece.creators.length; i++) {
                newPiece.creators.push(artPiece.creators[i]);
            }

            _mint(to, verbId);

            emit VerbCreated(verbId, artPiece);

            return verbId;
        }

```
In above recommendation removes the redundant variable creation, assigns and memory allocation which helps to save gas.

**Comparison Table**
| Feature  | Before  | After(Recommendated)   | More Efficient   |
|---|---|---|---|
| Line Count  | 15  | 12  | After
  |
| Variable Usage   | Uses `_artPiece` memory variable  | No additional memory variable  | After
  |
| Storage Access   | Reads artPiece memory variable repeatedly  | Reads from and writes to artPieces mapping directly  | After
  |
| Loops  | Loops through artPiece.creators array to copy data  | Loops through artPiece.creators array to copy data (assuming copy-on-write behavior)  | Both
  |
| Events Emitted  | Emits VerbCreated event  | mits VerbCreated event  | Both  |
| Gas Cost  | Potentially high due to memory reads and loop  | Possibly lower due to fewer operations  | After   |

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L292C1-L315C10



## 2. Remove redundant check to reduce the code size of the function:-
`_vote()` internal function which can be called by `vote()` public  while calling it set msg.sender as `address voter` it means voter cannot be be zero address. Therefore we can remove the check which reduce the code size of the function. Secondly called inside the `voteForManyWithSig()` external function but `_verifyVoteSignature()` verifies whether the `from` address is Zero or not.

```solidity
   function _vote(uint256 pieceId, address voter) internal {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        require(voter != address(0), "Invalid voter address"); // Gas no use
```

Recommendation:-
```solidity
   function _vote(uint256 pieceId, address voter) internal {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        //require(voter != address(0), "Invalid voter address"); // @audit check here.

```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L307C1-L309C63

## 3 . Redundant function can be cached to save gas :-
Private `parent()` function calls 3 times inside the while statement can cached to save gas and OPCODE.

Inside the `updateValue()` function
```solidity
 while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
                swap(position, parent(position));
                position = parent(position);
            }
```

Inside the `insert()` function
```solidity
        while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
            swap(current, parent(current));
            current = parent(current);
        }
```

Recommendation
```solidity
uint256 localParent = parent(position);
 while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[localParent]]) {
                swap(position, localParent);
                position = localParent;
            }


// insert()
uint localParent = parent(current) ;
while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[localParent]]) {
            swap(current, localParent);
            current = localParent;
        }

```
Our recommendation saves the usage of the opcode which continuously calls the `parent()` function.

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L125C1-L128C10
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L146C1-L149C14

## 4. `_depositPurchaseRewards()` function can be refactor to save gas :-
Inside the `_depositPurchaseRewards()` function first calls the `computePurchaseRewards()` function with `paymentAmountWei` argument to calculate the portion of the rewards should be distribute to the specified parties, inside this function calls `computeTotalReward()` with same `paymentAmountWei` variable but we can compute this inside the `_depositPurchaseRewards()` function only to reduce the code size of the contract will decrease the use of opcode make to save gas.Firstly we'll look into the `computePurchaseRewards()` function
```solidity
    function computePurchaseRewards(uint256 paymentAmountWei) public pure returns (RewardsSettings memory, uint256) {
        return (
            RewardsSettings({
                builderReferralReward: (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000,
                purchaseReferralReward: (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000,
                deployerReward: (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000,
                revolutionReward: (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000
            }),
            computeTotalReward(paymentAmountWei)
        );
    }

 //`_depositPurchaseRewards()` function

    function _depositPurchaseRewards(
        uint256 paymentAmountWei,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {
        (RewardsSettings memory settings, uint256 totalReward) = computePurchaseRewards(paymentAmountWei);

        if (builderReferral == address(0)) builderReferral = revolutionRewardRecipient;

        if (deployer == address(0)) deployer = revolutionRewardRecipient;

        if (purchaseReferral == address(0)) purchaseReferral = revolutionRewardRecipient;

        protocolRewards.depositRewards{ value: totalReward }(
            builderReferral,
            settings.builderReferralReward,
            purchaseReferral,
            settings.purchaseReferralReward,
            deployer,
            settings.deployerReward,
            revolutionRewardRecipient,
            settings.revolutionReward
        );

        return totalReward;
    }

```

Recommendation
```solidity

    function _depositPurchaseRewards(
        uint256 paymentAmountWei,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {

        if (builderReferral == address(0)) builderReferral = revolutionRewardRecipient;

        if (deployer == address(0)) deployer = revolutionRewardRecipient;

        if (purchaseReferral == address(0)) purchaseReferral = revolutionRewardRecipient;
        
        uint256 totalReward = computeTotalReward(paymentAmountWei);

        protocolRewards.depositRewards{ value: totalReward }(
            builderReferral,
            (paymentAmountWei * BUILDER_REWARD_BPS) / 10_000,
            purchaseReferral,
            (paymentAmountWei * PURCHASE_REFERRAL_BPS) / 10_000,
            deployer,
            (paymentAmountWei * DEPLOYER_REWARD_BPS) / 10_000,
            revolutionRewardRecipient,
            (paymentAmountWei * REVOLUTION_REWARD_BPS) / 10_000
        );

        return totalReward;
    }

```
Our recommendation does same job as `computePurchaseRewards()` function which could safely implement and it saves memory , opcode and gas.

**Comparison Table**
| Feature  | Before  | After(Recommendated)  | More efficient   |
|---|---|---|---|
| Variable Usage  | Stores RewardsSettings struc  | No additional memory usage  | After  |
| Calculations	  | Performs division twice per reward	  | Performs division once outside loop  | After  |
| Conditional checks  | Performs three conditional checks  | Performs three conditional checks	  | After  |
| Function calls	  | Calls `computePurchaseRewards()`   | No additional calls | After  |
| Gas cost  | Potentially higher due to calculations and function call  | Likely lower due to fewer operations  | After  |

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L65C1-L92C6
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L53C1-L64C6

## 5. `uint256 public quorumVotesBPS ` can be fit into the `uint16` and lesser than.
`quorumVotesBPS` varaible can be fit into the `uint16` and lesser than by looking into the `require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS)` this statement inside the `_setQuorumVotesBPS()` function which means `quorumVotesBPS` variable cannot exceed `10_000`.By making `uint256 public quorumVotesBPS` to `uint16 public quorumVotesBPS` can save `1 slot` which save 10_000 to 20_000 gas.

recommendation :-
```solidity
uint16 public quorumVotesBPS
```

code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L53C1-L54C35

## 6. Both for-loop can be combine :-
In the `batchVoteForManyWithSig()` function, two for-loops are used to verify signatures and vote for many pieces. These can be combined into a single loop, thereby reducing code size and gas execution.

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

ICultureIndex.ArtPiece memory AuctionArtPiece = verbs.getArtPieceById(_auction.verbId);

## 7. `_settleAuction()` can refactor to save gas :-
Else statement in the `_settleAuction()` calls the `getArtPieceById()` external function of VerbsToken contract two times as we know external calls were expensive and in for-loop it again calls `getArtPieceById()` fucntion with same arguments equaivalent to the length of creators array.So we can refactor by calling once and cached it utlised it to subsequent operation will be masssivley saves the execution gas.

```solidity
                //Total amount of ether going to creator
                uint256 creatorsShare = _auction.amount - auctioneerPayment; //30%
                uint256 numCreators = verbs.getArtPieceById(_auction.verbId).creators.length;
                address deployer = verbs.getArtPieceById(_auction.verbId).sponsor;

                //Build arrays for erc20TokenEmitter.buyToken
                uint256[] memory vrgdaSplits = new uint256[](numCreators);
                address[] memory vrgdaReceivers = new address[](numCreators);

                //Transfer auction amount to the DAO treasury
                _safeTransferETHWithFallback(owner(), auctioneerPayment);

                uint256 ethPaidToCreators = 0;

                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                if (creatorsShare > 0 && entropyRateBps > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = verbs.getArtPieceById(_auction.verbId).creators[i];
                        vrgdaReceivers[i] = creator.creator;
                        vrgdaSplits[i] = creator.bps;

                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
                    }
                }

``` 

Recommendation:-
```solidity

 		//Total amount of ether going to creator
                uint256 creatorsShare = _auction.amount - auctioneerPayment; //30%
                ICultureIndex.ArtPiece memory AuctionArtPiece = verbs.getArtPieceById(_auction.verbId); //@audit check here
                uint256 numCreators = AuctionArtPiece.creators.length;//@audit check here
                address deployer = AuctionArtPiece.sponsor;//@audit check here

                //Build arrays for erc20TokenEmitter.buyToken
                uint256[] memory vrgdaSplits = new uint256[](numCreators);
                address[] memory vrgdaReceivers = new address[](numCreators);

                //Transfer auction amount to the DAO treasury
                _safeTransferETHWithFallback(owner(), auctioneerPayment);

                uint256 ethPaidToCreators = 0;

                //Transfer creator's share to the creator, for each creator, and build arrays for erc20TokenEmitter.buyToken
                if (creatorsShare > 0 && entropyRateBps > 0) {
                    for (uint256 i = 0; i < numCreators; i++) {
                        ICultureIndex.CreatorBps memory creator = AuctionArtPiece.creators[i];//@audit check here
                        vrgdaReceivers[i] = creator.creator;
                        vrgdaSplits[i] = creator.bps;
                        //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);
                    }
                }
```
Please look into the `//@audit check here` comment in the above code. We can assure that above recommended code will saves `SLAODS` and external calls execution gas which is safe to implement also.

**Comparison Table**
| Feature  | Before  | After  |
|---|---|---|
| Storage reads  | 4+  | 1  |
| Gas cost  | High  | Low  |
| Auditability  | Less auditable  | More auditable  |
| Maintainability  | Less maintainable	  | More maintainable  |


Code snippet:-
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L367C16-L388C1