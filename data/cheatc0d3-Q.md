# Revolution Protocal QA Report

## Summary

This report presents the findings of a smart contract audit performed on RevolutionProtocol's CultureIndex contract focused on security. The audit examined the contract source code for commonly known and more obscure security vulnerabilities.

### Parameter Validation

Function parameters are not validated which could cause errors or unintended states.

Recommendation: Mark appropriate variables as immutable.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L209

```solidity
function createPiece(ArtPieceMetadata calldata metadata, CreatorBps[] calldata creators) external {

    require(bytes(metadata.name).length > 0, "Name is required");
    require(bytes(metadata.description).length > 0, "Description is required");
    require(creators.length > 0, "At least one creator is required");
    
    // Rest of function
}
```

### Use Immutable State Variables 

Using immutable declares variables that cannot be changed after contract deployment, saving gas.

Recommendation: Mark appropriate variables as immutable.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L78

```solidity
// Mark as immutable
address public immutable dropperAdmin;  
IRevolutionBuilder private immutable manager;
```


### Add Reentrancy Guards

External calls in the voting logic could enable reentrancy attacks.

Recommendation: Apply nonReentrant modifier.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L332

```solidity
// Use reentrancy guard for vote logic
function vote(uint256 pieceId) nonReentrant public {
    // Voting logic
}

```

### Emit events for important actions

Adding events for key actions improves visibility and enables off-chain services.

Recommendation: Emit events on initialization, votes, etc.



https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L332

```solidity
event VoteCast(address voter, uint256 pieceId); 

function vote(uint256 pieceId) external {
   // Logic

   emit VoteCast(msg.sender, pieceId)
}
```

### Use interface for external contract interactions:

An interface decouples dependencies on external contracts.

Recommendation: Create ERC20 interface to use.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L265

```
interface IERC721 {
  function balanceOf(address owner) external view returns (uint256);
}

IERC721 erc721Contract = IERC721(_erc721VotingToken);

function getVotes(address account) public view returns (uint256) {
  uint256 erc721Balance = erc721Contract.balanceOf(account);

  // Rest of logic  
}
```

This prevents issues in case the ERC721 contract changes.

### Use pull over push model:

Retrieve external data within functions reduces attack surface.

Recommendation: Call external contracts internally over accepting external input.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L332

```
function vote(uint256 pieceId) external nonReentrant {
  uint256 weight = erc20VotingToken.getVotes(msg.sender);
  _vote(pieceId, msg.sender, weight);  
}

function _vote(uint256 pieceId, address voter, uint256 weight) internal {
  // Internal logic  
}
```

Retrieve data within contract instead of accepting externally.

### Initialize variables:

Uninitialized structs can cause unexpected errors.

Recommendation: Explicitly initialize variables.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol#L209

```
ArtPieceMetadata memory metadata; 
CreatorBps[] memory creatorArray;

function createPiece(/* params */) public {
  metadata = ArtPieceMetadata({
    // Initialize struct fields
  });

  // Set creator array

  // Create piece logic
}
```

This prevents uninitialized storage errors.

### Lack of events for critical state changes

The contract does not emit events for changes, reducing transparency for users and off-chain services.

Recommendation: Emit events on all externally accessible state changes.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L136

The `updateValue()` function lacks events for transparency into changes:

```diff
function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
+   emit ValueUpdated(itemId, newValue);

    // Update value mapping
    valueMapping[itemId] = newValue;
} 
```

### Lack of input validation

Lack of validation on new heap values being inserted enables bad data to cause failures.

Recommendation: Check inputs meet requirements.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L136

The `updateValue()` function lacks input validation on the new value passed in:

```diff
function updateValue(uint256 itemId, uint256 newValue) public onlyAdmin {
+  require(newValue >= min && newValue <= max, "Out of range");
  
  valueMapping[itemId] = newValue; 
}
```

### Missing upper bound check

Inserting new items lacks a check that the heap is under maximum capacity.

Recommendation: Check size on insert to prevent errors.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L119


```diff
function insert(uint256 itemId, uint256 value) public onlyAdmin {
+ require(size < maxSize, "Heap full");

  heap[size] = itemId;
  size++;
}
```

### Missing event emissions

The MaxHeap contract does not emit events for critical state changes like `insert`, `updateValue`, and `extractMax`. This reduces transparency into how the heap is being modified.

**Recommendation:** Emit events on all externally-facing state changes:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L136
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L156

```diff
+ event HeapInsert(uint256 indexed itemId, uint256 value);

+ event HeapUpdate(uint256 indexed itemId, uint256 oldValue, uint256 newValue);  

+ event HeapExtract(uint256 indexed itemId, uint256 value);

function insert(uint256 itemId, uint256 value) public onlyInserter {
+  emit HeapInsert(itemId, value);
  
  // Insert logic
}

function updateValue(uint256 itemId, uint256 newValue) public onlyUpdater { 
+  emit HeapUpdate(itemId, valueMapping[itemId], newValue);

  // Update logic  
}

function extractMax() public onlyExtractor returns (uint256, uint256) {
+ uint256 itemId = heap[0];
+ uint256 value = valueMapping[itemId]; 
+ emit HeapExtract(itemId, value); 

  // Extract logic

  return (itemId, value);
}
```

### Lack of error handling in external calls

The MaxHeap constructor calls the `IRevolutionBuilder` manager contract but does not check the return value. If the call fails, it could lead to unexpected behavior.

**Recommendation:** Check return values from external calls:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L31

```diff 
constructor(address _manager) payable initializer {
-   manager = IRevolutionBuilder(_manager);
+   IRevolutionBuilder revolutionBuilder = IRevolutionBuilder(_manager);
+   require(address(revolutionBuilder) != address(0), "Invalid manager address");   
+   manager = revolutionBuilder;
}
```

### Cache pointers to reduce mapping lookups

Caching pointers and optimizing data layout can significantly reduce gas costs.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L94

```solidity
uint256[] childPointers; 

function maxHeapify(uint256 pos) internal {
  // Use cached child pointers
  uint256 left = childPointers[pos*2];  
  uint256 right = childPointers[pos*2 + 1];
  
  //... rest of function
}

function insert(uint256 itemId, uint256 value) public onlyAdmin {

  // Update child pointer cache 
  childPointers.push(itemId);
  
  // ... rest of function
}

```


### Mark fields immutable 

Declaring immutable state saves gas by storing in code vs storage.

Recommendation: Mark admin & manager fields immutable.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L16

```solidity
address public immutable admin;
IRevolutionBuilder private immutable manager;

// Wrap heap updates to return new instances
function insert(uint item, uint value) public returns (Heap memory) {
   Heap storage heap = heaps[msg.sender]; 
   // Copy-on-write logic
   Heap memory newHeap = Heap(heap); 
   newHeap.insert(item, value);
   heaps[msg.sender] = newHeap;
   return newHeap;
}
```

### Consider Adding Bounds Checking

Insert and heapify lack checks against max size and invalid positions.

Recommendation: Add require statements validating critical parameters.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L119

```solidity
function insert(uint256 itemId, uint256 value) public onlyAdmin {

  require(size < heap.length, "Heap is full");

  // Insert logic  
}

function maxHeapify(uint256 pos) internal {

  require(pos < size, "Invalid position");
  
  // Heapify logic
}
```

### Deleting Items Functionality missing

Support to remove items from the heap is currently missing.

Recommendation: Implement a delete function to enable removing items.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol

```solidity 
function remove(uint256 itemId) public onlyAdmin {

  uint256 position = positionMapping[itemId];
  
  // Move the last element to this position
  heap[position] = heap[size-1]; 
  
  // Correct mappings
  positionMapping[heap[size-1]] = position;
  
  // Heapify
  maxHeapify(position);
  
  // Decrement size
  size--;
}
```

### Memory structure improvements

Storing items separately from heap ids saves gas.

Recommendation: Refactor Item struct and storage layout.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L119

```solidity
struct Item {
  uint256 value;
  uint256 id;
}

// Store items separately from heap structure
mapping(uint256 => Item) public items;
uint256[] heap; 

function insert(uint256 id, uint256 value) public {

  Item memory item = Item(value, id);

  items[id] = item;

  // Insert id into heap
}
```

### Reentrancy protection

Extract logic calls external contracts without protection.

Recommendation: Add reentrancy guard to sensitive functions.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L119

```solidity
// Shared lock
ReentrancyGuard sharedLock;  

function insert(uint256 id, uint256 value) public nonReentrant(sharedLock) {

    // Critical section protected by mutex
    sharedLock.lock();

    // Insert logic

    sharedLock.unlock();
}
```

### Use SafeMath library for overflow protected math:

Unchecked math could enable overflow attacks.

Recommendation: Use SafeMath library for overflow safe operations.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L119

```
using SafeMath for uint256; 

function insert(uint256 itemId, uint256 value) public onlyAdmin {

  require(itemId != 0, "itemId cannot be 0");
  require(value > 0, "value must be > 0");

  // Prevent overflow
  if(size.add(1) > type(uint256).max) {
    revert(); 
  }

  size = size.add(1);
  
  // Other logic
}

function getParent(uint256 pos) private pure returns (uint256) {
  // Overflow check
  if(pos == 0 || pos == 1) {
    revert(); 
  }

  return pos.sub(1).div(2);
}

```

### Add reentrancy guards:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L156

```
function extractMax() nonReentrant external onlyAdmin returns (uint256, uint256) {

  require(size > 0, "Heap empty");
  
  // Interactions & state changes
  uint256 popped = heap[0]; 

  heap[0] = heap[--size];
  maxHeapify(0);

  // Returns only after all changes
  return (popped, valueMapping[popped]); 
}
```

### Validate return parameters:

Returned values from functions lack sanity checks.

Recommendation: Validate return parameters meet requirements.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L169

```
function getMax() external view returns (uint256, uint256) {

  (uint256 item, uint256 value) = super.getMax();

  // Validate return value
  require(value > 0, "Invalid getMax value"); 

  return (item, value);
}
```

### Validate critical contract addresses:

Constructor accepts manager address without validity check.

Recommendation: Require address is not zero address.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/MaxHeap.sol#L169

```
constructor(address _manager) {

  require(_manager != address(0), "Invalid manager address");
  manager = IRevolutionManager(_manager);

}
```

### Missing slippage protection in functions

Front-running attacks could rapidly modify balances/supply without limit.

Recommendation: Implement min/max change thresholds based on past values.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L101

**Before:**
```solidity
// No slippage protection in functions
function _transfer(address from, address to, uint256 value) internal override {
    revert TRANSFER_NOT_ALLOWED();
}
```

**After:**
```solidity
// Add slippage protection in functions
function _transfer(address from, address to, uint256 value) internal override {
    require(value <= _getMaxSlippage(from, to), "ERC20: slippage protection");
    revert TRANSFER_NOT_ALLOWED();
}
```

### Validate invalid amount on mint:

Zero or negative amounts can be minted leading to undesired states.

Recommendation: Validate amount parameters are positive integers.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L134

```
function mint(address account, uint256 amount) public onlyOwner {

  require(amount > 0, "Invalid amount");

  _mint(account, amount);
}
```

This prevents invalid amounts.

### Add reentrancy protection:

External calls in functions require protection to prevent reentrancy attacks.

Recommendation: Apply nonReentrant modifier to sensitive functions.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol#L134

```
function mint(address account, uint256 amount) public nonReentrant onlyOwner {
  // Logic
}
```

### Checking return values:

Transfer and send calls lack checking return value for success.

Recommendation: Validate return value instead of relying on revert on failure.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L152

```solidity
function buyToken(...) internal {

  // ...

  (bool success,) = treasury.call{value: toPayTreasury}("");

  require(success, "Transfer failed");

}
```

### Validate token address on initialize:

The token address is not checked for the zero address on initialization.

Recommendation: Require valid contract address is passed.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L84 

```
function initialize(/* params */) external onlyInitializing {

  require(_erc20Token != address(0), "Invalid token address");
  
  // Rest of logic
}
```

This prevents setting the token to the zero address.

### Use SafeMath for overflow protection:

Unchecked math operations could potentially overflow.

Recommendation: Utilize SafeMath library for overflow safety.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L152

```
using SafeMath for uint256;

function buyToken(/* params */) public payable {

  uint256 toPayTreasury = msgValueRemaining.mul(10000 - creatorRateBps).div(10000);
  
  // Rest of logic
}
```

### Add reentrancy guards around external calls:

External calls in functions require protection against reentrancy.

Recommendation: Apply nonReentrant modifier to sensitive functions.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L152

```
function buyToken(/* params */) public nonReentrant {

  uint256 treasuryPayment = computePaymentAmount();
  
  bool success;
  {
    nonReentrant;
    success = treasury.call{value: treasuryPayment}("");
  }
  require(success, "Transfer failed");
  
  // Rest of logic  
}
```

### Validate token minting:

Minting tokens lacks validation on destination address and amount parameters.

Recommendation: Implement input validation in mint function.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/ERC20TokenEmitter.sol#L108

```
function _mint(address account, uint256 amount) private {

  require(account != address(0), "Invalid address");
  require(amount > 0, "Invalid amount");
  
  token.mint(account, amount);
}
```
### Custom bid failure errors:

Revert errors lack context into failure reasons for debugging.

Recommendation: Implement custom errors providing context on failure conditions.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L171

```solidity
function createBid(uint256 verbId, address bidder) external payable override {

  if (bidder == address(0)) {
    revert BidderCannotBeZeroAddress(); 
  }

  if (_auction.verbId != verbId) {
    revert InvalidAuctionedVerb(verbId, _auction.verbId);
  }
  
  // ... rest of logic

}

error BidderCannotBeZeroAddress();
error InvalidAuctionedVerb(uint256 actualId, uint256 expectedId);
```

### Validate critical addresses:

The contract accepts token addresses without validation in initialize.

Recommendation: Require addresses are not zero address on initialize.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L113

```
function initialize(
  address _verbs, 
  address _erc20Token
  // other params
) external onlyInitializing {

  require(_verbs != address(0), "Invalid verbs token");
  require(_erc20Token != address(0), "Invalid erc20 token");
  
  // Rest of logic
}
```

### Use pull over push model: 

Relying on external call for withdrawals introduces reentrancy risk.

Recommendation: Replace external call with internal transfer logic.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/AuctionHouse.sol#L171

```
function createBid(uint256 verbId, address bidder) external payable nonReentrant {

  uint256 bid = msg.value;
  address lastBidder = auction.bidder;

  // Internal logic

  if (lastBidder != address(0)) {
    _withdraw(lastBidder, auction.amount); 
  }  

}

function _withdraw(address to, uint256 amount) internal {
  // transfer eth
}
```

This prevents reentrancy with custom withdraw logic instead of rely on call.

### Custom error codes:

Standard reverts lack context into failure reasons for tracing bugs.

Recommendation: Implement custom errors providing more context.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L273

```solidity
function getArtPieceById(uint256 verbId) public view returns (ArtPiece memory) {

  if (verbId > _currentVerbId) {
    revert InvalidTokenId(verbId); 
  }

  return artPieces[verbId];

}

error InvalidTokenId(uint256 verbId);
```


### Checked math:

Unchecked math could potentially overflow without notifications.

Recommendation: Use checked math operations for overflows.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L273

```solidity
function _mintTo(address to) internal returns (uint256) {

  uint256 verbId = _currentVerbId;

  unchecked {
    ++_currentVerbId;  
  }

  // Rest of minting logic
  
  return verbId;

}
```

### Use interface for external contract interactions:

Direct external contract reliance amplifies changes risk.

Recommendation: Interact through well defined interfaces.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L281

```
interface ICultureIndex {
  function dropTopVotedPiece() external returns (ArtPiece memory); 
}

ICultureIndex cultureIndex = ICultureIndex(cultureIndexAddress);

function _mintTo(address to) internal {

  cultureIndex.dropTopVotedPiece();
  
  // Rest of logic
}
```

This also prevents issues if CultureIndex contract changes. 

### Validate critical variables:

Lack of input validation could enable bad input triggering undesired states.

Recommendation: Implement sanity checks on parameters.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol#L281

```
function _mintTo(address to) internal {

  require(to != address(0), "Invalid mint address");
  
  // Get top voted piece

  require(piece.creators.length <= MAX_CREATORS, "Too many creators");

  // Mint logic  
}
```


### Use custom errors

Specific errors aid debugging and failure handling.

Recommendation: Use custom errors for common failure cases.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/VerbsToken.sol

```
error DropFailed(string reason);

if (/* drop failed */) {
  revert DropFailed("No pieces available"); 
}
```
### Improve Comments to be more detailed:

Formula explanations aid readability and maintenance.

Recommendation: Expand math formula comments.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L85

```solidity
// p(x) represents the price function where:
// p0 = targetPrice 
// k = priceDecayPercent
// x = tokens sold
// r = perTimeUnit
function pIntegral(int256 timeSinceStart, int256 sold) internal view returns (int256) {

  // pIntegral calculates the integral of p(x), which represents 
  // the cumulative sum of (tokens sold * price) at each point

  // Formula is:
  // ∫p0 * (1 - k)^(x/r) dx  

  // Implemented as:
  return 
    (targetPrice * perTimeUnit) / 
    decayConstant * 
    (
      (1 - priceDecayPercent)^timeSinceStart - 
      (1 - priceDecayPercent)^(timeSinceStart - (sold / perTimeUnit))
    );

}
```

### Validating parameters:

Inputs lack validation potentially enabling invalid parameters.

Recommendation: Implement sanity checks on arguments.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L28

```solidity
constructor(
  int256 _targetPrice,
  int256 _priceDecayPercent, 
  int256 _perTimeUnit
) {

  require(_priceDecayPercent >= 0 && _priceDecayPercent <= 90 * WAD, "Invalid decay percent");
  
  require(_perTimeUnit >= 0, "Tokens per unit must be positive");

  // Rest of initialization
}
```


### Use SafeMath for overflow/underflow protection:

Unchecked math could trigger overflows without notification.

Recommendation: Utilize SafeMath for overflow protection.

```
using SafeMath for uint; 

function xToY(/* params */) public view returns (int256) {

  sold + amount; // Safe against overflows

  // Other math
}
```

### Validate inputs to prevent invalid parameters:

Malformed inputs could lead to undesired states.

Recommendation: Add require checks on parameters.

```
function xToY(/* params */) public view {

  require(timeSinceStart >= 0, "Invalid time");
  require(sold >= 0, "Invalid sold amount");
  require(amount > 0, "Invalid amount");
    
}
```

### Use custom errors with reason strings:

Custom errors improve failure tracing and handling.

Recommendation: Use custom errors for common failure scenarios.

```
error InvalidTime(string reason);

if(timeSinceStart < 0) {
  revert InvalidTime("Time cannot be negative"); 
}
```

### Follow best practices pattern

Adhering to checks-effects-interactions improves reasonability.

Recommendation: Validate, calculate, externalize access.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/libs/VRGDAC.sol#L47

```
function xToY(/* params */) public view {

  // Input validation

  // Safe math calculations 
  
  return computedValue;
}
```
### Adding a pause: 

Allowing pausing distributions helps handle emergencies.

Recommendation: Implement owner controlled pause functionality.

```solidity
bool public paused;

modifier whenNotPaused() {
  require(!paused, "Contract paused");
  _;
}

function pause() external onlyOwner {
  paused = true;
}

function handleRewards() public whenNotPaused {
  // Logic
}
```

### Validate addresses on initialization:

Addresses are not checked for zero address validity.

Recommendation: Require valid contract addresses on initialize.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L7

```
constructor(
  address _protocolRewards, 
  address _revolutionRewardRecipient
) {

  require(_protocolRewards != address(0), "Invalid protocol rewards address");
  require(_revolutionRewardRecipient != address(0), "Invalid recipient address");

  // Initialization logic  
}
```

### Use SafeMath for overflow protection: 

Unchecked math could trigger overflows silently.

Recommendation: Utilize SafeMath for overflow safety.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12

```
using SafeMath for uint256;

function _handleRewardsAndGetValueToSend(
  uint256 msgValue, 
  /* other params */
) internal returns (uint256) {

  uint256 rewards = computeTotalReward(msgValue);

  require(msgValue > rewards, "Invalid ETH amount");
  
  // Other logic
}
```

### Follow checks-effects-interactions pattern:

Adhere to checks-effects-interactions pattern.

Recommendation: Validate, compute, call external.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/TokenEmitter/TokenEmitterRewards.sol#L12

```
function _handleRewardsAndGetValueToSend(
  uint256 msgValue,
  /* other params */  
) internal returns (uint256 valueToSend) {

  // Input validation
  
  // Safe computations

  return valueToSend;
}
```
### Use SafeMath for overflow protection: 

Unchecked math could trigger overflows without notification.

Recommendation: Utilize SafeMath for arithmetic safety.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40

```
using SafeMath for uint256;

function computeTotalReward(uint256 paymentAmount) public pure returns (uint256) {

  uint256 builderReward = paymentAmount.mul(BUILDER_REWARD_BPS).div(10_000);

  // Avoid overflows
}
```

### Validate critical input data:

Lack of input validation allows bad data causing failures.

Recommendation: Implement parameter sanity checks.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L40

```
function computeTotalReward(uint256 paymentAmount) public pure {
  

  require(paymentAmount >= minPurchaseAmount);
  require(paymentAmount <= maxPurchaseAmount);

  // Logic
}
```

### Follow checks-effects-interactions pattern:

Adhere to checks-effects-interactions pattern.

Recommendation: Validate, compute, externalize access.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/protocol-rewards/src/abstract/RewardSplits.sol#L66

```
function _depositPurchaseRewards(/* params */) internal {

  // Input validation
  
  // Local computations 
  
  // Deposit rewards 
  protocolRewards.depositRewards{value: totalReward}(/* params */);

}
```