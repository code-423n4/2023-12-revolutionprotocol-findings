## [GAS - 01] Pack storage to consume one slot instead of 2 slots at struct `ICultureIndex.CreatorBps`

- `ICultureIndex.CreatorBps` => https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L98-L101

```solidity
    struct CreatorBps {
        address creator;
--->    uint256 bps;
    }
```

- We could have been stored the above struct in one slot by making `bps` as type `uint96`.
- Since the `bps` ids limited to `<= 10000`, it is safe to make it type `uint96`
- check the below change to make this gas savings.

### recommendation

```diff
    struct CreatorBps {
        address creator;
-       uint256 bps;
+       uint96 bps;
    }
```



## [GAS - 02] wasting gas to make check, which is always safe at [CultureIndex.voteForManyWithSig]

- `CultureIndex.voteForManyWithSig ` => https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L375-L377

```solidity
    function voteForManyWithSig(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external nonReentrant {
--->    bool success = _verifyVoteSignature(from, pieceIds, deadline, v, r, s);
--->    if (!success) revert INVALID_SIGNATURE();

        _voteForMany(pieceIds, from);
    }
```

- In the above code, the returned `bool success` from the internal `_verifyVoteSignature` call, the bool will always be success or will revert inside that internal function itself. (See the internal function below)

```solidity
    function _verifyVoteSignature(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal returns (bool success) {
        require(deadline >= block.timestamp, "Signature expired");

        bytes32 voteHash;

        voteHash = keccak256(abi.encode(VOTE_TYPEHASH, from, pieceIds, nonces[from]++, deadline));

        bytes32 digest = _hashTypedDataV4(voteHash);

        address recoveredAddress = ecrecover(digest, v, r, s);

        // Ensure to address is not 0
        if (from == address(0)) revert ADDRESS_ZERO();

        // Ensure signature is valid
        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE();

--->    return true;
    }
```

- So, it makes the validation of `success` variable, a waste operation to spend the gas.

### recommendation

```diff
    function voteForManyWithSig(
        address from,
        uint256[] calldata pieceIds,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external nonReentrant {
-       bool success = _verifyVoteSignature(from, pieceIds, deadline, v, r, s);
-       if (!success) revert INVALID_SIGNATURE();

+       _verifyVoteSignature(from, pieceIds, deadline, v, r, s);

        _voteForMany(pieceIds, from);
    }
```



## [GAS - 02] No need to check `if (msgValue < computeTotalReward(msgValue))`

- `TokenEmitterRewards._handleRewardsAndGetValueToSend ` => https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12-L21

```solidity
    function _handleRewardsAndGetValueToSend(
        uint256 msgValue,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {
--->    if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();

        return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
    }
```

- `if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();`
- The right hand side `computeTotalReward` will always be equal to 2.5% of the msg.value (which is left hand side of the validation).

```solidity
    function computeTotalReward(uint256 paymentAmountWei) public pure returns (uint256) {
        if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();

        return
            (paymentAmountWei * BUILDER_REWARD_BPS) /
            10_000 +
            (paymentAmountWei * PURCHASE_REFERRAL_BPS) /
            10_000 +
            (paymentAmountWei * DEPLOYER_REWARD_BPS) /
            10_000 +
            (paymentAmountWei * REVOLUTION_REWARD_BPS) /
            10_000;
    }
```

- So, it just adds all the fees of builder, purchaser, deployer and revolution reward recipient which totally equals 2.5% of the `paymentAmountWei` input param.
- So  it is a waste operation to do `if (msgValue < computeTotalReward(msgValue))`.

### recommendation

```diff
    function _handleRewardsAndGetValueToSend(
        uint256 msgValue,
        address builderReferral,
        address purchaseReferral,
        address deployer
    ) internal returns (uint256) {
-       if (msgValue < computeTotalReward(msgValue)) revert INVALID_ETH_AMOUNT();

        return msgValue - _depositPurchaseRewards(msgValue, builderReferral, purchaseReferral, deployer);
    }
```
