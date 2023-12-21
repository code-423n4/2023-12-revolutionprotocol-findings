# Low Risk Issues

| ID |Title|
|:--:|:-------|
|[L-01]| Edge case bug in `SignedWadMath.wadMul` |
|[L-02]| Incorrect `require` statement in `VerbsToken.sol:getArPieceById` |
|[L-03]| Incorrect calculation in `ERC20TokenEmitter.sol:getTokenQuoteForPayment` |

## [L-01] Edge case bug in `SignedWadMath.wadMul`

The code of the `SignedWadMath` library contains a bug that causes the `wadMul` function to return incorrect result in the edge case for `a * b` where `a == -1` and `b == min int256`. Although this edge case does not seem possible in the current implementation, it would be recommended to modify the library with the [bug fix](https://github.com/transmissions11/solmate/pull/380).

## [L-02] Incorrect `require` statement in `VerbsToken.sol:getArPieceById`

`VerbsToken.sol:getArPieceById` will not revert if `verbId == _currentVerbId` is passed as an argument, being `_currentVerbId` the next ID to be minted.

```solidity
File: VerbsToken.sol
273    function getArtPieceById(uint256 verbId) public view returns (ICultureIndex.ArtPiece memory) {
274        require(verbId <= _currentVerbId, "Invalid piece ID"); 👈 Should be `verbId < _currentVerbId`
275        return artPieces[verbId];
    }
```

The protocol currently calls this function only from `AuctionHouse.sol` and will not pass invalid IDs. However, it would be recommended to add the check to avoid potential issues in third-party contracts or future protocol upgrades.

## [L-03] Incorrect calculation in `ERC20TokenEmitter.sol:getTokenQuoteForPayment`

The calculation of the tokens emitted for a payment amount in `ERC20TokenEmitter.sol:getTokenQuoteForPayment` does not account for the share of purchase amount [sent to the `creatorAddress`](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L196), which will result in an overestimation of the tokens emitted.

```solidity
File: ERC20TokenEmitter.sol
266    /**
267     * @notice Returns the amount of tokens that would be emitted for the payment amount, taking into account the protocol rewards.
268     * @param paymentAmount the payment amount in wei.
269     * @return gainedX The amount of tokens that would be emitted for the payment amount.
270     */
271    function getTokenQuoteForPayment(uint256 paymentAmount) external view returns (int gainedX) {
272        require(paymentAmount > 0, "Payment amount must be greater than 0");
273        // Note: By using toDaysWadUnsafe(block.timestamp - startTime) we are establishing that 1 "unit of time" is 1 day.
274        // solhint-disable-next-line not-rely-on-time
275        return
276            vrgdac.yToX({
277                timeSinceStart: toDaysWadUnsafe(block.timestamp - startTime),
278                sold: emittedTokenWad,
279                amount: int(((paymentAmount - computeTotalReward(paymentAmount)) * (10_000 - creatorRateBps)) / 10_000) 👈 Does not include `creatorDirectPayment`
280            });
281    }
```


# Non-Critical Issues

| ID |Title|Instances|
|:--:|:-------|:--:|
|[N-01]| Incorrect comments | 3 |
|[N-02]| Unnecessary checks | 4 |

## [N-01] Incorrect comments

```solidity
File: CultureIndex.sol
147    ///                                                          ///
148    ///                         MODIFIERS                        /// 👈 No modifiers in this file
149    ///                                                          ///
```

```solidity
File: CultureIndex.sol
269    /** 
270     * @notice Returns the voting power of a voter at the current block. 👈 Should be `at a specific block` and add a `@param blockNumber`
271     * @param account The address of the voter.
272     * @return The voting power of the voter.
273     */
274     function getPastVotes(address account, uint256 blockNumber) external view override returns (uint256) {
```

```solidity
File: VerbsToken.sol
173    /**
174     * @notice Mint a Verb to the minter.
175     * @dev Call _mintTo with the to address(es). 👈 Should be `Call _mintTo with the minter address`
176     */
177    function mint() public override onlyMinter nonReentrant returns (uint256) {
```

## [N-02] Unnecessary checks

```solidity
File: CultureIndex.sol
375        bool success = _verifyVoteSignature(from, pieceIds, deadline, v, r, s);
376
377        if (!success) revert INVALID_SIGNATURE(); 👈 `_verifyVoteSignature` cannot return false
```

```solidity
File: CultureIndex.sol
403        for (uint256 i; i < len; i++) {
404            if (!_verifyVoteSignature(from[i], pieceIds[i], deadline[i], v[i], r[i], s[i])) revert INVALID_SIGNATURE(); 👈 `_verifyVoteSignature` cannot return false
405        }
```

```solidity
File: CultureIndex.sol
438        if (from == address(0)) revert ADDRESS_ZERO();
439
440        // Ensure signature is valid
441        if (recoveredAddress == address(0) || recoveredAddress != from) revert INVALID_SIGNATURE(); 👈 `recoveredAddress == address(0)` already checked above
```

```solidity
File: CultureIndex.sol
486    function topVotedPieceId() public view returns (uint256) {
487        require(maxHeap.size() > 0, "Culture index is empty"); 👈 Checked by `maxHeap.getMax()`
488        //slither-disable-next-line unused-return
489        (uint256 pieceId, ) = maxHeap.getMax();
```