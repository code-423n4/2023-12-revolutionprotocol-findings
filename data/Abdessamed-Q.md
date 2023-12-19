- The address of `WETH` stored in [AuctionHouse](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L54) can be set to `constant` as it is non-upgradable contract

- The check `require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");` inside [_setQuorumVotesBPS](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L499) is contradictory with the function's doc `* @dev newQuorumVotesBPS must be greater than the hardcoded min`
```solidity
/**
* @notice Admin function for setting the quorum votes basis points
* @dev newQuorumVotesBPS must be greater than the hardcoded min
* @param newQuorumVotesBPS new art piece drop threshold
*/
function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external onlyOwner {
     // audit-info Contradictory check with the function's doc
     require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");
      emit QuorumVotesBPSSet(quorumVotesBPS, newQuorumVotesBPS);

      quorumVotesBPS = newQuorumVotesBPS;
}
```
