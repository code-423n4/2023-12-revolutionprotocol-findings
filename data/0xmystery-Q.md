## `ECDSA.recover` over `ecrecover`
One of the most critical aspects to note about `ecrecover` is its vulnerability to malleable signatures. This means that a valid signature can be transformed into a different valid signature without needing access to the private key. Where possible, adopt `ECDSA.recover` as commented by the imported `EIP712Upgradeable` in CultureIndex.sol.

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/cryptography/EIP712Upgradeable.sol#L93-L110

```solidity
    /**
     * @dev Given an already https://eips.ethereum.org/EIPS/eip-712#definition-of-hashstruct[hashed struct], this
     * function returns the hash of the fully encoded EIP712 message for this domain.
     *
     * This hash can be used together with {ECDSA-recover} to obtain the signer of a message. For example:
     *
     * ```solidity
     * bytes32 digest = _hashTypedDataV4(keccak256(abi.encode(
     *     keccak256("Mail(address to,string contents)"),
     *     mailTo,
     *     keccak256(bytes(mailContents))
     * )));
     * address signer = ECDSA.recover(digest, signature);
     * ```
     */
    function _hashTypedDataV4(bytes32 structHash) internal view virtual returns (bytes32) {
        return MessageHashUtils.toTypedDataHash(_domainSeparatorV4(), structHash);
    }
``` 
Here's a specific instance entailed:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L435

```solidity
        address recoveredAddress = ecrecover(digest, v, r, s);
```
## Unutilized function
`CultureIndex.hasVoted`:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L250-L258

```solidity
    /**
     * @notice Checks if a specific voter has already voted for a given art piece.
     * @param pieceId The ID of the art piece.
     * @param voter The address of the voter.
     * @return A boolean indicating if the voter has voted for the art piece.
     */
    function hasVoted(uint256 pieceId, address voter) external view returns (bool) {
        return votes[pieceId][voter].voterAddress != address(0);
    }
``` 
could have been used in the following require statement:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L311

```diff
-        require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
+        require(!(hasVoted(pieceId, voter)), "Already voted");
```

## Typo mistakes
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L64

```diff
-    /// @notice Struct to represent an item in the heap by it's ID
+    /// @notice Struct to represent an item in the heap by its ID
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L64-L65

```diff
-    /// @notice Struct to represent an item in the heap by it's ID
    mapping(uint256 => uint256) public heap;
+    /// @notice Mapping to represent an item in the heap by it's ID
    mapping(uint256 => uint256) public heap;
```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L437-L438

```diff
-        // Ensure to address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();
+        // Ensure from address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();
```