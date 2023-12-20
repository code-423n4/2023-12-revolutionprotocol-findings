### Report 1:
#### Unnecessary code complexities
As edited below one of the condition statement in the _vote(...) function uses "!" operator twice in other to ensure only address that has not voted before are allowed to vote, instead of using two negatives, they can simply cancel out each other, "==" operator should be used as it makes the code more readable and direct
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L311
```solidity
 function _vote(uint256 pieceId, address voter) internal {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        require(voter != address(0), "Invalid voter address");
        require(!pieces[pieceId].isDropped, "Piece has already been dropped");
---        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
+++        require( votes[pieceId][voter].voterAddress == address(0 ), "Already voted");
 ...
```
### Report 2:
#### Wrong Comment Description
The code implementation in the code implementation in the onlyAdmin() modifier does not correlate to the comment description in any way, the comment description seems to be a requirement for a function implementation and not relevant to the modifier.
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L39
```solidity
    /**
>>>     * @notice Require that the minter has not been locked.
     */
    modifier onlyAdmin() {
        require(msg.sender == admin, "Sender is not the admin");
        _;
    }
```
### Report 3:
#### Irreversible lock Functions
Purposely having an Irreversible lock function for minter is totally wrong and not user friendly. Since the Owner is given the power to lock minter, a function to unlock in event of needed circumstances is of upmost importance
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L221
```solidity
    /**
     * @notice Lock the minter.
     * @dev This cannot be reversed and is only callable by the owner when not locked.
     */
    function lockMinter() external override onlyOwner whenMinterNotLocked {
        isMinterLocked = true;

        emit MinterLocked();
    }
+++   function UnlockMinter() external override onlyOwner {
+++        isMinterLocked = false;

+++        emit MinterUnLocked();
+++    }
```
Other instances of this vulnerability can be found at [L240](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L240) & [L260](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L260)
### Report 4:
#### DoS Risk in Zero transfer Event
The buyToken(...) function call as provided below shows a transfer of "toPayTreasury" value through call transfer, though the chances of the value of  "toPayTreasury" being zero is very low that does not still rule out the possibility, therefore a validation should be done as precaution to ensure call transfer is only called if  "toPayTreasury" is not zero. The sensitivity of the need for this can be seen in how transfer of creatorDirectPayment  is handled with the necessary non zero validation in the same function.
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L191
```solidity
function buyToken(
        address[] calldata addresses,
        uint[] calldata basisPointSplits,
        ProtocolRewardAddresses calldata protocolRewardsRecipients
    ) public payable nonReentrant whenNotPaused returns (uint256 tokensSoldWad) {
        ...
        //Deposit funds to treasury
+++   if (toPayTreasury > 0) {
        (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
        require(success, "Transfer failed.");
+++    }

        //Transfer ETH to creators
        if (creatorDirectPayment > 0) {
            (success, ) = creatorsAddress.call{ value: creatorDirectPayment }(new bytes(0));
            require(success, "Transfer failed.");
        }
 ...
```
### Report 5:
#### Incomplete Code Implementation
As noted in the code below necessary security considerations should be added based on block created to prevent flash attacks.
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L321
```solidity
function _vote(uint256 pieceId, address voter) internal {
        require(pieceId < _currentPieceId, "Invalid piece ID");
        require(voter != address(0), "Invalid voter address");
        require(!pieces[pieceId].isDropped, "Piece has already been dropped");
        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");

        uint256 weight = _getPastVotes(voter, pieces[pieceId].creationBlock);
        require(weight > minVoteWeight, "Weight must be greater than minVoteWeight");

        votes[pieceId][voter] = Vote(voter, weight);
        totalVoteWeights[pieceId] += weight;

        uint256 totalWeight = totalVoteWeights[pieceId];

>>>        // TODO add security consideration here based on block created to prevent flash attacks on drops?
        maxHeap.updateValue(pieceId, totalWeight);
        emit VoteCast(pieceId, voter, weight, totalWeight);
    }
```
### Report 6:
#### Unnecessary Validation
From the code provided below a validation is done to check if "msgValue" is less than "computeTotalReward(msgValue)", the problem with this is that there is no way this can ever be possible as computeTotalReward(...) returns 2.5% of "msgValue", so the validation check is not necessary.
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18
```solidity
   function _handleRewardsAndGetValueToSend(
        uint256 msgValue,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {
>>>        if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();

        return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
    }
```