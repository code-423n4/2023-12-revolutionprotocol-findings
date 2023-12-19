# Low-Risk Findings

## L-01 - `RewardSplits::computeTotalReward` reverts when using the min and max purchase amounts

Inside `RewardSplits`, the following constants are defined:

```solidity
uint256 public constant minPurchaseAmount = 0.0000001 ether;
uint256 public constant maxPurchaseAmount = 50_000 ether;
```

[RewardSplits.sol#L23-L24](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L23-L24)

One would assume that these are the minimum and maximum valid amounts for making a purchase.

However, `RewardSplits::computeTotalReward` reverts if these amounts are used:

```solidity
if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

[RewardSplits.sol#L41](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41)

### Impact

If someone queries the purchase amount limits and tries to use them, the transaction will revert. Instead, the minimum and maximum valid amounts are `minPurchaseAmount + 1` and `maxPurchaseAmount - 1`, respectively.

### Recommended Mitigation Steps

Since the current behavior can lead to confusion, I recommend accepting the minimum and maximum amounts as defined in the contract:

```diff
- if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
+ if (paymentAmountWei < minPurchaseAmount || paymentAmountWei > maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

## L-02 - `CultureIndex::createPiece` accepts pieces with empty names

Although the comment on the `createPiece` function specifies that “metadata must include name, description, and image”, sponsor confirmed that only a `name` should be enforced.

```solidity
/**
 ...
 * Requirements:
 * - `metadata` must include name, description, and image. Animation URL is optional.
 ...
 */
function createPiece(
    ArtPieceMetadata calldata metadata,
    CreatorBps[] calldata creatorArray
) public returns (uint256) { ... }
```

[CultureIndex.sol#L204-L212](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L204-L212)

The current implementation is not making any validations on the `metadata.name`.

### Impact

Art pieces with empty names in the metadata can be created.

### Recommended Mitigation Steps

Add the appropriate validations on the `metadata.name`, inside the `createPiece` function.

# Non-Critical Findings

## NC-01 - `VRGDAC::pIntegral` differs from the reference implementation

In the [reference implementation by transmissions11](https://gist.github.com/transmissions11/485a6e2deb89236202bd2f59796262fd#file-vrgdac-L73-L81), `pIntegral` represents the indefinite integral of the [linear VRGDA formula](https://www.paradigm.xyz/2022/08/vrgda#:~:text=the%20VRGDA%20formula.-,Linear,-Let%E2%80%99s%20say%20we) (w.r.t `n`).

$$
p(n) = p_0 (1-k)^{t - \frac{n}{r}}
$$

However, in [Revolution’s implementation](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L86-L96), `pIntegral(int256 t, int256 n)` represents the integral of the linear VRGDA in (0, n].

`pIntegral` is only used inside `VRGDAC::xToY`, which is implemented the same as the reference implementation:

```solidity
function xToY(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {
    unchecked {
        return pIntegral(timeSinceStart, sold + amount) - pIntegral(timeSinceStart, sold);
    }
}
```

[VRGDAC.sol#L47-L51](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L47-L51)

Here, the original implementation is computing:

$$
y=\int_n^{n+x} p(z) \text{d}z
$$

where `n = sold` and `x = amount`.

On the other hand, Revolution’s implementation is computing:

$$
y = \int_0^{n+x} p(z) \text{d}z - \int_0^{n} p(z) \text{d}z
$$

**The end result `y` will be the same in both cases, although the original implementation is more efficient**.

### Recommended Mitigation Steps

Replace the `pIntegral` function with the reference implementation by transmissions11, to avoid confusion and reduce gas usage.

## NC-02 - `VRGDAC::yToX` differs from the reference implementation

In the [reference implementation by transmissions11](https://gist.github.com/transmissions11/485a6e2deb89236202bd2f59796262fd#file-vrgdac-L61-L71), the `yToX` function is implemented as follows:

```solidity
function yToX(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {
    unchecked {
        return wadMul(
            -wadDiv(
                wadLn(1e18 - wadMul(amount, wadDiv(decayConstant, wadMul(perTimeUnit, p(timeSinceStart, sold))))),
                decayConstant
            ),
            perTimeUnit
        );
    }
}
```

In math notation:

$$
x_1=-r\frac{\ln(1-y\frac{d}{p(t,n)r})}{d}
$$

where:

- `decayConstant` = $\ln(1-k) = d$
- `amount` = $y$
- `perTimeUnit` = $r$
- `sold` = $n$
- `timeSinceStart` = $t$
- `priceDecayPercent` = $k$
- `targetPrice` = $p_0$
- `p(t, n)` = $p_0(1-k)^{t - \frac{n}{r}}$

On the other hand, Revolution’s implementation looks like this:

```solidity
function yToX(int256 timeSinceStart, int256 sold, int256 amount) public view virtual returns (int256) {
    int256 soldDifference = wadMul(perTimeUnit, timeSinceStart) - sold;
    unchecked {
        return
            wadMul(
                perTimeUnit,
                wadDiv(
                    wadLn(
                        wadDiv(
                            wadMul(
                                targetPrice,
                                wadMul(
                                    perTimeUnit,
                                    wadExp(wadMul(soldDifference, wadDiv(decayConstant, perTimeUnit)))
                                )
                            ),
                            wadMul(
                                targetPrice,
                                wadMul(
                                    perTimeUnit,
                                    wadPow(1e18 - priceDecayPercent, wadDiv(soldDifference, perTimeUnit))
                                )
                            ) - wadMul(amount, decayConstant)
                        )
                    ),
                    decayConstant
                )
            );
    }
}
```

[VRGDAC.sol#L54-L83](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L54-L83)

In math notation:

$$
x_2 = r\frac{\ln(\frac{num}{den})}{d}
$$

with $num$ and $den$ being:

$$
num = p_0 r \exp((rt-n)\frac{d}{r})
$$

$$
den = p_0r(1-k)^{(rt-n)/r} - y d
$$

### Proof that both implementations are the same

Let’s prove the reference implementation and Revolution’s implementation are the same (ie. $x_1=x_2$).

Remember $d=\ln(1-k)$, so $\exp$ and $\ln$ cancel out:

$$
\exp((rt-n)\frac{d}{r}) = \exp(d)^{(rt-n)/r}=(1-k)^{\frac{(rt-n)}{r}}=(1-k)^{t-\frac{n}{r}}
$$

$num$ can then be simplified to be:

$$
num = p_0 r (1-k)^{(rt-n)/r}
$$

Thus we have:

$$
(\frac{num}{den})^{-1}=\frac{den}{num}= 1 - y\frac{d}{p(t,n)r}
$$

Finally we see that $x_2=x_1$:

$$
x_2 = r\frac{\ln(\frac{num}{den})}{d}=-r\frac{\ln((\frac{num}{den})^{-1})}{d} = -r\frac{\ln(1-y\frac{d}{p(t,n)r})}{d} = x_1
$$

### Recommended Mitigation Steps

Since we saw that both implementations are equivalent, replace the `yToX` function with the reference implementation by transmissions11, to avoid confusion and reduce gas usage.