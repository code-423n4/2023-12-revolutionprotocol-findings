## Revolution Protocal Gas Report

**VRGDA.sol**

### Precompute static values in VRGDA

Static values that do not change can be precomputed to save gas.

Recommendation: Compute static values like LN_ONE_MINUS_DECAY on deployment.

```solidity
// VRGDA.sol

uint256 private constant LN_ONE_MINUS_DECAY = ln(1e18 - priceDecayPercent);

function pIntegral(int256 timeSinceStart, int256 sold) internal view returns (int256) {   

  return (targetPrice * perTimeUnit) / decayConstant * 
    (
      (1e18 - priceDecayPercent)**timeSinceStart -  
      (1e18 - priceDecayPercent)**(timeSinceStart - (sold / perTimeUnit))
    );

}
```

**TokenEmitterRewards.sol**

### Consolidate calldata in TokenEmitterRewards

Grouping common parameters reduces ABI encoding/decoding costs.

Recommendation: Pass data as structs instead of individual values.

```solidity
// TokenEmitterRewards.sol

struct RewardsReceivers {
  address builder;
  address referrer;
  address deployer; 
  address revolution;
}

function depositRewards(RewardsReceivers memory receivers) external {

  protocolRewards.deposit{value: amount}(
    receivers // pass struct 1 time instead of each address
  )

}
```

**VerbsToken.sol** 

### Cache state locally in VerbsToken

Reading state variables has higher gas cost than local vars.

Recommendation: Copy state vars into local vars before usage.

```solidity 
// VerbsToken.sol

function _mintVerb() internal {

  uint256 verbId = _currentTokenId; // local var

  _currentTokenId++;  

  _mint(msg.sender, verbId);

}
```

**CultureIndex.sol**

### Optimization opportunities in CultureIndex

The `vote()` and `_vote()` functions require multiple external contract calls to the ERC20 and ERC721 contracts to check balances and vote weights. This can be gas intensive.

Recommendation: Introduce vote weight cache mapping to reduce external calls.

```solidity
// Cache voting weights in mapping to avoid external contract calls
mapping(address => uint96) public voteWeights;

// Update vote weight cache on voting token transfers
function updateVoteWeightCache(address voter) internal {
   voteWeights[voter] = _calculateVoteWeight(
      erc20.balanceOf(voter),
      erc721.balanceOf(voter)
   );
}
```