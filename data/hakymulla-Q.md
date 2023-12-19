## [01] Different Implementation compared to comment
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L499

The `CultureIndex::_setQuorumVotesBPS` function has the comments
```
     /**
     * @notice Admin function for setting the quorum votes basis points
     * @dev newQuorumVotesBPS must be greater than the hardcoded min
```
But implemented newQuorumVotesBPS is lesser than or equal to the maximum settable quorum votes basis points

```
   require(newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "CultureIndex::_setQuorumVotesBPS: invalid quorum bps");
```

## [02] Solidity version might not work in L2 chain

All contract are compiled in Solidity version 0.8.20

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L2

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L18

In Solc compiler version 0.8.20, the default target EVM version has been switched to Shanghai. This alteration includes PUSH0 opcodes in the generated bytecode. 

Checking optimisim contracts and they use pragma solidity >0.5.0 <0.8.0 and others pragma solidity 0.8.15;

Failure to select the appropriate EVM version may result in deployment issues for your contracts.