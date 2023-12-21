# [QA-01]  `minPurchaseAmount` and `maxPurchaseAmount` should not be inclusive

[File: RewardSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L41)
```
if (paymentAmountWei <= minPurchaseAmount || paymentAmountWei >= maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

`paymentAmountWei` should not be smaller than `minPurchaseAmount` and should not be greater than `maxPurchaseAmount`.
However, the current implementation defines, that function will also revert when `paymentAmountWei == minPurchaseAmount` or `paymentAmountWei == maxPurchaseAmount`.
Our recommendation is to changed `>=` and `<=` to `>` and `<`:

```
if (paymentAmountWei < minPurchaseAmount || paymentAmountWei > maxPurchaseAmount) revert INVALID_ETH_AMOUNT();
```

# [QA-02] Misleading change in the OpenZeppelin code-base


`NontransferableERC20Votes` inherits from `ERC20VotesUpgradeable` which inherits from `ERC20Upgradeable`.
When we look at `base/erc20/ERC20VotesUpgradeable.sol` and `base/erc20/ERC20Upgradeable.sol` - we can notice, that the code-base was basically copy-pasted from the Open Zeppelin.
However, there's small change in `ERC20Upgradeable.sol`. In the OZ's implementation `_mint()` is not virtual. However, `base/erc20/ERC20Upgradeable.sol` implements it as virtual.

This leads to the situation, when `_mint()` can be (and is) overriden in `NontransferableERC20Votes.sol`

[File: NontransferableERC20Votes.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/NontransferableERC20Votes.sol#L125)
```
  * NOTE: This function is not virtual, {_update} should be overridden instead.
     */
    function _mint(address account, uint256 value) internal override {
        if (account == address(0)) {
            revert ERC20InvalidReceiver(address(0));
        }
        _update(address(0), account, value);
    }
```

This may lead to some confusion - especially, that NatSpec is copy-pasted from OZ's implementation and states that: `This function is not virtual, {_update} should be overridden instead.`

* Function's `_mint()` NatSpec states `This function is not virtual` - but it's virtual
* `ERC20Upgradeable` assumes that `_mint()` should not be virtual, but it's virtual and `NontransferableERC20Votes.sol` overrides it.


# [R-01] Name of the `onlyMinter` modifier and `minter` variable in `VerbsToken.sol` should be changed

[File: VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L96)
```
 /**
     * @notice Require that the sender is the minter.
     */
    modifier onlyMinter() {
        require(msg.sender == minter, "Sender is not the minter");
        _;
    }
```

According to NatSpec and modifier's name, the `onlyMinter` modifier should be used on `mint()` function. It checks if the sender has minter capabilities.

However, this modifier is not only used on `mint()` function:

```
function mint() public override onlyMinter nonReentrant returns (uint256) {
```

but also in a `burn()` function:

```
function burn(uint256 verbId) public override onlyMinter nonReentrant {
```

Our suggestion is to change `minter` variable to `minterOrBurner` and `onlyMinter` modifier to `onlyMinterOrBurner`. The name of the variable and the modifier will straightforwardly point that this modifier is used in both `mint()` and `burn()`. The current name suggests that it's used only for minting.

# [R-02] Changing critical parameters should emit an event with both old and new value

[File: VerbsToken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L209-L214)
```
 function setMinter(address _minter) external override onlyOwner nonReentrant whenMinterNotLocked {
        require(_minter != address(0), "Minter cannot be zero address");
        minter = _minter;

        emit MinterUpdated(_minter);
    }
```

`MinterUpdated()` should emit both old and new value.

The same issue occurs in `setCultureIndex()`, `setDescriptor()`, `setMinter()` from `VerbsToken.sol`
The same issue occurs in `setCreatorRateBps()`, `setMinCreatorRateBps()`, `setEntropyRateBps()`, `setTimeBuffer()` from `AuctionHouse.sol`


# [R-03] Functions which updates the value do not verify if the value was really changed

Whenever we update/set a new value of some state variable, it's a good practice to make sure that, that value is being indeed updated.
In the current implementation of `setCreatorRateBps()` in `AuctionHouse.sol`, even when the `_creatorRateBps` won't be changed, function will still emit an `CreatorRateBpsUpdated()` event - which might be misleading to the end-user.
E.g. let's assume that `_creatorRateBps = 100`. Calling `setCreatorRateBps(100)` won't update/set anything new, but `CreatorRateBpsUpdated()` will still be called.

Our recommendation is to add additional `require` check, to verify, that provided values are indeed different than the current ones:

```
require(_creatorRateBps != creatorRateBps, "Nothing to update");
```


The same issue occurs in `setCreatorRateBps()`, `setMinCreatorRateBps()`, `setEntropyRateBps()`, `setTimeBuffer()` from `AuctionHouse.sol`
The same issue occurs in `setCultureIndex()`, `setDescriptor()`, `setMinter()` from `VerbsToken.sol`


# [R-04] Use modifier in `MaxHeap.sol` instead of repeating the same code

In `MaxHeap.sol`, both `extractMax()` and `getMax()` implements below check: `require(size > 0, "Heap is empty");`.
Instead of repeating the same code twice, we can implement this check in a modifier (e.g. heapNotEmpty) and use that modifier on `getMax()` and `extractMax()`.


# [R-05] `MAX_QUORUM_VOTES_BPS` is constant and cannot be updated

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L48)
```
 uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%
```

The maximum settable quorum votes basis points is set to a constant value. There's no function which allows `onlyOwner` to update this value. Our suggestion is to change this value from constant to non-constant and implement additional function which will allow `onlyOwner` to update `MAX_QUORUM_VOTES_BPS`.

# [R-06] `MAX_NUM_CREATORS` is constant and cannot be updated

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L75)
```
 uint256 public constant MAX_QUORUM_VOTES_BPS = 6_000; // 6,000 basis points or 60%
```

The maximum number of creators is set to a constant value. There's no function which allows `onlyOwner` to update this value. Our suggestion is to change this value from constant to non-constant and implement additional function which will allow `onlyOwner` to update `MAX_NUM_CREATORS`.

# [NC-01] Use underscore to better represent number of zeros in numbers

[File: RewardSplits.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/protocol-rewards/src/abstract/RewardSplits.sol#L23-L24)
```
    uint256 public constant minPurchaseAmount = 0.0000001 ether;
    uint256 public constant maxPurchaseAmount = 50_000 ether;
```

While code uses `_` to improve readability of large numbers (e.g. `50_000`), this can be also used in `0.0000001`:

```
uint256 public constant minPurchaseAmount = 0.000_000_1 ether;
```

# [N-02] Incorrect grammar

[File: VerbsTOken.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L41)
```
 // An address who has permissions to mint Verbs
```

`address who` should be changed to `address which`.

# [N-03] Less complicated expressions should be used first in `if` conditions

[File: MaxHeap.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L102C15-L102C45)
```
if (pos >= (size / 2) && pos <= size) return;
```

FIrstly, this will improve code readability. Secondly, because of the Solidity short-circuiting behavior we can save some gas. In expression: `A() AND B()` - whenever `A()` returns `false`, `B()` won't be executed (because no matter of the `B()` value - `false AND whatever` evaluates to `false`).

`pos <= size` - uses only `<=` operation
`pos >= (size / 2)` - uses division operation and then `>=` operation

This implies, that above condition should be rewritten to:

```
if (pos <= size && pos >= (size / 2)) return;
```

# [N-04] Redundant double negation

[File: CultureIndex.sol](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L311)
```
require(!(votes[pieceId][voter].voterAddress != address(0)), "Already voted");
```

Using double negation makes the code more complex and wastes gas. Above line is the same as:

```
require(votes[pieceId][voter].voterAddress == address(0), "Already voted");
```