### [N-1] Function state mutability can be restricted to pure

**Description:** `NontransferableERC20Votes::_transfer` state mutability can be restricted to pure
We don't read any storage variables, only use the arguments therefore, it can be restricted to pure.

<details>
<summary>Line of code</summary>

```solidity
101:    function _transfer(address from, address to, uint256 value) internal override {
```

</details>

**Recommended Mitigation:** Set function state mutability to pure

```diff
-    function _transfer(address from, address to, uint256 value) internal override {
+    function _transfer(address from, address to, uint256 value) internal pure override {
```

### [N-2] Function state mutability can be restricted to pure

**Description:** `NontransferableERC20Votes::_approve` state mutability can be restricted to pure
We don't read any storage variables, only use the arguments therefore, it can be restricted to pure.

<details>
<summary>Line of code</summary>

```solidity
141:    function _approve(address owner, address spender, uint256 value) internal override {
```

</details>

**Recommended Mitigation:** Set function state mutability to pure

```diff
-    function _approve(address owner, address spender, uint256 value) internal override {
+    function _approve(address owner, address spender, uint256 value) internal pure override {

```