## [L-1] invoke all init functions of all parents of  upgradeable contracts
The init functions of all parent contracts of VerbsToken.sol are not invoke. As best practice, it is important to invoke all init functions of parent contracts in an upgradeable design.

According to proxy-based upgradeability standard, constructors are only needed to initialize `  immutable `  variables, hence the need for regular functions to initialize state variables conventionally named ` initialize `  function.

However, while Solidity ensures that all constructors are called only once during deployment, the developer has to ensure calling the initialize functions before the use of a proxy contract. 

Solidity ensures the call to all parent contract constructors but for proxy-based upgradeable contract, the developer has to ensure all initialize functions of the the parent functions are also called.

The VerbsToken contract inherits from `  ERC721CheckpointableUpgradeable `  but the ` __ERC721Votes_init()`  function is not invoked in the initialize function of VerbsToken contract.

```````````````` function __ERC721Votes_init() internal onlyInitializing {}`  

  ERC721CheckpointableUpgradeable `   also inherit from ` VotesUpgradeable `  that also has a ` __Votes_init()`   init function that was never invoked.

`` function __Votes_init() internal onlyInitializing {}` 

The ` VotesUpgradeable `  also inherit from ` EIP712Upgradeable `  which also has an init function that was never invoked.

` function __EIP712_init(string memory name, string memory version) internal onlyInitializing {
        __EIP712_init_unchained(name, version);
    }` 


Files: 
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L33C10-L40C2
- https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L130C5-L156C6

## [L-2] Wrong comment in _verifyVoteSignature function of CultureIndex.sol

The comment reads "Ensure to address is not 0" instead of "Ensure from address is not 0"

`````` // Ensure to address is not 0 //@audit-info this comment is wrong: replace to with from
        if (from == address(0)) revert ADDRESS_ZERO();`   

File: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L437C9-L438C55

## [L-3] Ensure ` paymentAmount `  is greater than zero before sending ETH 
In some situations  ` paymentAmount `   can be zero so there is a need to check if   ` paymentAmount `   and not try to send zero ETH afterwards.

``` //Calculate paymentAmount for specific creator based on BPS splits - same as multiplying by creatorDirectPayment
                        uint256 paymentAmount = (creatorsShare * entropyRateBps * creator.bps) / (10_000 * 10_000);//@audit div before mul
                        ethPaidToCreators += paymentAmount;

                        //Transfer creator's share to the creator//@audit gas: if amout is zero dont send
                        _safeTransferETHWithFallback(creator.creator, paymentAmount);`


File: https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L389C24-L394C86

