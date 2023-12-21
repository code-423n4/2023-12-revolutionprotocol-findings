# G-1. OR in if-condition can be rewritten to two single if conditions #

Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

Reference : https://code4rena.com/reports/2023-08-shell#g-05-or-in-if-condition-can-be-rewritten-to-two-single-if-conditions

Instances :
```
File : packages/revolution/src
/CultureIndex.sol

441 :         if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L441
```
File : packages/revolution/src
/AuctionHouse.sol

268 :         if (auction.startTime == 0 || auction.settled) {
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L268
```
File :  packages/protocol-rewards/src/abstract
/RewardSplits.sol

30 :         if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");

41 :         if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L30

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41

# G-2. Using a positive conditional flow to save a NOT opcode #

Estimated savings: 3 gas per Instance.

Reference : https://code4rena.com/reports/2023-07-basin#g-13-using-a-positive-conditional-flow-to-save-a-not-opcode

Instances :
```
File :  packages/revolution/src
/AuctionHouse.sol

433 :         if (!success) {

441 :             if (!wethSuccess) revert("WETH transfer failed");

454 :         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L433

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L441

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L454
```
File : packages/revolution/src
/CultureIndex.sol

377 :         if (!success) revert INVALID_SIGNATURE();

404 :             if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE();

545 :         if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L377

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L404

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L545

# G-3. Using bools for storage incurs overhead #

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.

Reference : https://code4rena.com/reports/2023-07-basin#g-11---using-bools-for-storage-incurs-overhead

Instances : 
```
File : packages/revolution/src/ERC20TokenEmitter.sol

191 :         (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L191
```
File : packages/revolution/src/CultureIndex.sol

375 :         bool success = _verifyVoteSignature(from, pieceIds, deadline, v, r, s);
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L375

# G-4. Variables: “constants” expressions are expressions, not constants #

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

If the variable was immutable instead: the calculation would only be done once at deploy time (in the constructor), and then the result would be saved and read directly at runtime rather than being recalculated.

Consequences: each usage of a “constant” costs ~100gas more on each access (it is still a little better than storing the result in storage, but not much..). since these are not real constants, they can’t be referenced from a real constant environment (e.g. from assembly, or from another library )

Change these expressions from constant to immutable and implement the calculation in the constructor or hardcode these values in the constants and add a comment to say how the value was calculated.

Reference : https://github.com/ethereum/solidity/issues/9232
```
FIle : packages/revolution/src/CultureIndex.sol

29 - 30 :     bytes32 public constant VOTE_TYPEHASH =
        keccak256("Vote(address from,uint256[] pieceIds,uint256 nonce,uint256 deadline)");
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L29C4-L31C1

# G-5. abi.encode() is less efficient than abi.encodepacked() #

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Reference : https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison
```
File : packages/revolution/src/CultureIndex.sol

441 :         voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L431

# G-6. With assembly, .call (bool success) transfer can be done gas-optimized #

Return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
(bool success,) = dest.call{value:amount}("");
bool success;
assembly {

success := call(gas(), dest, amount, 0, 0)
}  

Reference : https://code4rena.com/reports/2023-07-basin#g-04-with-assemblycall-bool-successtransfer-can-be-done-gas-optimized
```
File : packages/revolution/src/ERC20TokenEmitter.sol

191 :         (bool success, ) = treasury.call{ value: toPayTreasury }(new bytes(0));
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L191
