# Title - gas opt #1
## Link to affected code
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L286

## Rating / Impact
Gas optimization.

## Observation
Check not required, since this condition is already checked in the ["CultureIndex" contract](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L182).

If you believe it is necessary to address this concern to mitigate the risk that an upgrade to 'CultureIndex' may no longer include such a check, please also evaluate other conditions. For instance, verify whether the sum of the creator's basis points adds up to 10'000. Failure to ensure this condition poses a risk of locking the AuctionHouse contract, as the contract no longer verifies this aspect.

## Recommended mitigation steps
Consider removing the referenced check above.

# Title - gas opt #2
## Link to affected code
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L385

## Rating / Impact
Gas optimization.

## Observation
The 'creator' value is requested again in each loop iteration leading to unnecessary gas consumation.

## Recommended mitigation steps
Consider obtaining the 'creator' once outside of the loop.

# Title - gas opt #3
## Link to affected code
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12

## Rating / Impact
Gas optimization.

## Observation
The rewards calculations are done multiple times leading to more gas consumption than actually necessary.

For instace, when calling [_handleRewardsAndGetValueToSend](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12) rewards are calculated 3 times:

* https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L18 calling: 
    * https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40
* https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L20
    * https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L56
    * https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L62

## Recommended mitigation steps
Consider removing redundant calculations to save gas.