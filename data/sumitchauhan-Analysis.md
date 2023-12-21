REVOLUTION PROTOCOL ANALYSIS REPORT


Medium Issues

1.
The owner is a single point of failure and a centralization risk:
  ERC20TokenEmitter.sol::288 => function setEntropyRateBps(uint256 _entropyRateBps) external onlyOwner {
  ERC20TokenEmitter.sol::299 => function setCreatorRateBps(uint256 _creatorRateBps) external onlyOwner {

----------------------------------------------------------------------------------------------------------------------
  
Low Issues

1.
Contract locking ether found: 
NontransferableERC20Votes.sol::44 => Contract NontransferableERC20Votes has payable functions:
- NontransferableERC20Votes.constructor(address) 
  But does not have a function to withdraw the ether

2.
Unsafe ERC20 Operation(s):
  ERC721Upgradeable.sol::188 => transferFrom(from, to, tokenId);
  AuctionHouse.sol::361 => else verbs.transferFrom(address(this), _auction.bidder, _auction.verbId);

3.
Owner can renounce Ownership:
  OwnableUpgradeable.sol::94 => function renounceOwnership() public virtual onlyOwner {

-------------------------------------------------------------------------------------------------------------------------    
Informational Issue

1.
Non efficient zero initialization:
  ERC20TokenEmitter.sol::205 => uint256 bpsSum = 0;
  ERC20TokenEmitter.sol::209 => for (uint256 i = 0; i < addresses.length; i++) { 

2.
Constants in comparisons should appear on the left side:
  VotesUpgradeable.sol::221 => if (from == address(0)) {
  VotesUpgradeable.sol::224 => if (to == address(0)) {
  VotesUpgradeable.sol::235 => if (from != to && amount > 0) {
  VotesUpgradeable.sol::236 => if (from != address(0)) {
  VotesUpgradeable.sol::244 => if (to != address(0)) {
  ERC721Upgradeable.sol::83 => if (owner == address(0)) {
  ERC721Upgradeable.sol::166 => if (to == address(0)) {
  ERC721Upgradeable.sol::236 => if (owner == address(0)) {
  ERC721Upgradeable.sol::277 => if (auth != address(0)) {
  ERC721Upgradeable.sol::282 => if (from != address(0)) {
  ERC721Upgradeable.sol::291 => if (to != address(0)) {
  ERC721Upgradeable.sol::317 => if (to == address(0)) {
  ERC721Upgradeable.sol::321 => if (previousOwner != address(0)) {
  ERC721Upgradeable.sol::362 => if (previousOwner == address(0)) {
  ERC721Upgradeable.sol::379 => if (to == address(0)) {
  ERC721Upgradeable.sol::383 => if (previousOwner == address(0)) {
  ERC721Upgradeable.sol::443 => if (emitEvent || auth != address(0)) {
  ERC721Upgradeable.sol::447 => if (auth != address(0) && owner != auth && !isApprovedForAll(owner, auth)) {
  ERC721Upgradeable.sol::469 => if (operator == address(0)) {
  ERC721Upgradeable.sol::484 => if (owner == address(0)) {
  ERC721Upgradeable.sol::500 => if (to.code.length > 0) {
  ERC721Upgradeable.sol::506 => if (reason.length == 0) {
  AuctionHouse.sol::120 => require(msg.sender == address(manager), "Only manager can initialize");
  AuctionHouse.sol::121 => require(_weth != address(0), "WETH cannot be zero address");
  AuctionHouse.sol::175 => require(bidder != address(0), "Bidder cannot be zero address");
  AuctionHouse.sol::195 => if (lastBidder != address(0)) _safeTransferETHWithFallback(lastBidder, _auction.amount);
  AuctionHouse.sol::222 => require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
  AuctionHouse.sol::235 => require(_minCreatorRateBps <= 10_000, "Min creator rate must be less than or equal to 10_000");
  AuctionHouse.sol::254 => require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");
  AuctionHouse.sol::268 => if (auction.startTime == 0 || auction.settled) {
  AuctionHouse.sol::339 => require(_auction.startTime != 0, "Auction hasn't begun");
  AuctionHouse.sol::350 => if (_auction.bidder != address(0)) {
  AuctionHouse.sol::358 => if (_auction.bidder == address(0))
  AuctionHouse.sol::363 => if (_auction.amount > 0) {
  AuctionHouse.sol::383 => if (creatorsShare > 0 && entropyRateBps > 0) {
  ERC721EnumerableUpgradeable.sol::98 => if (previousOwner == address(0)) {
  ERC721EnumerableUpgradeable.sol::103 => if (to == address(0)) {
  ERC721EnumerableUpgradeable.sol::193 => if (amount > 0) {  
  VRGDAC.sol::38 => require(decayConstant < 0, "NON_NEGATIVE_DECAY_CONSTANT");
  RewardSplits.sol::30 => if (_protocolRewards == address(0) || _revolutionRewardRecipient == address(0)) revert("Invalid Address Zero");
  RewardSplits.sol::74 => if (builderReferral == address(0)) builderReferral = revolutionRewardRecipient;
  RewardSplits.sol::76 => if (deployer == address(0)) deployer = revolutionRewardRecipient;
  RewardSplits.sol::78 => if (purchaseReferral == address(0)) purchaseReferral = revolutionRewardRecipient;
  ERC20Upgradeable.sol::192 => if (from == address(0)) {
  ERC20Upgradeable.sol::195 => if (to == address(0)) {
  VotesUpgradeable.sol::235 => if (from != to && amount > 0) {
  VotesUpgradeable.sol::236 => if (from != address(0)) {
  VotesUpgradeable.sol::244 => if (to != address(0)) {
  VotesUpgradeable.sol::221 => if (from == address(0)) {
  ERC20TokenEmitter.sol::91 => require(msg.sender == address(manager), "Only manager can initialize");
  ERC20TokenEmitter.sol::96 => require(_treasury != address(0), "Invalid treasury address");
  ERC20TokenEmitter.sol::160 => require(msg.value > 0, "Must send ether");
  ERC20TokenEmitter.sol::188 => if (totalTokensForCreators > 0) emittedTokenWad += totalTokensForCreators;
  ERC20TokenEmitter.sol::195 => if (creatorDirectPayment > 0) {
  ERC20TokenEmitter.sol::201 => if (totalTokensForCreators > 0 && creatorsAddress != address(0)) {
  ERC20TokenEmitter.sol::210 => if (totalTokensForBuyers > 0) {
  ERC20TokenEmitter.sol::217 => require(bpsSum == 10_000, "bps must add up to 10_000");
  ERC20TokenEmitter.sol::238 => require(amount > 0, "Amount must be greater than 0");
  ERC20TokenEmitter.sol::255 => require(etherAmount > 0, "Ether amount must be greater than 0");
  ERC20TokenEmitter.sol::272 => require(paymentAmount > 0, "Payment amount must be greater than 0");
  ERC20TokenEmitter.sol::289 => require(_entropyRateBps <= 10_000, "Entropy rate must be less than or equal to 10_000");
  ERC20TokenEmitter.sol::300 => require(_creatorRateBps <= 10_000, "Creator rate must be less than or equal to 10_000");
  ERC20TokenEmitter.sol::310 => require(_creatorsAddress != address(0), "Invalid address");
  ERC20TokenEmitter.sol::91 => require(msg.sender == address(manager), "Only manager can initialize");
  NontransferableERC20Votes.sol::128 => if (account == address(0)) { 
  OwnableUpgradeable.sol::56 => if (initialOwner == address(0)) {
  OwnableUpgradeable.sol::103 => if (newOwner == address(0)) {
  ERC1967Upgrade.sol::53 => if (_data.length > 0 || _forceCall) {
  MaxHeap.sol::56 => require(msg.sender == address(manager), "Only manager can initialize");
  MaxHeap.sol::79 => require(pos != 0, "Position should not be zero");
  MaxHeap.sol::157 => require(size > 0, "Heap is empty");
  MaxHeap.sol::170 => require(size > 0, "Heap is empty");
  NonTrnasferableERC20Votes.sol::1668 => if (value >> 128 > 0) {
  NonTrnasferableERC20Votes.sol::1672 => if (value >> 64 > 0) {
  NonTrnasferableERC20Votes.sol::1676 => if (value >> 32 > 0) {
  NonTrnasferableERC20Votes.sol::1680 => if (value >> 16 > 0) {
  NonTrnasferableERC20Votes.sol::1684 => if (value >> 8 > 0) {
  NonTrnasferableERC20Votes.sol::1688 => if (value >> 4 > 0) {
  NonTrnasferableERC20Votes.sol::1692 => if (value >> 2 > 0) {
  NonTrnasferableERC20Votes.sol::1696 => if (value >> 1 > 0) {
  NonTrnasferableERC20Votes.sol::1721 => if (value >= 10 ** 64) {
  NonTrnasferableERC20Votes.sol::1725 => if (value >= 10 ** 32) {
  NonTrnasferableERC20Votes.sol::1729 => if (value >= 10 ** 16) {
  NonTrnasferableERC20Votes.sol::1733 => if (value >= 10 ** 8) {
  NonTrnasferableERC20Votes.sol::1737 => if (value >= 10 ** 4) {
  NonTrnasferableERC20Votes.sol::1741 => if (value >= 10 ** 2) {
  NonTrnasferableERC20Votes.sol::1745 => if (value >= 10 ** 1) {
  NonTrnasferableERC20Votes.sol::1772 => if (value >> 128 > 0) {
  NonTrnasferableERC20Votes.sol::1776 => if (value >> 64 > 0) {
  NonTrnasferableERC20Votes.sol::1780 => if (value >> 32 > 0) {
  NonTrnasferableERC20Votes.sol::1784 => if (value >> 16 > 0) {
  NonTrnasferableERC20Votes.sol::1788 => if (value >> 8 > 0) {
  NonTrnasferableERC20Votes.sol::2371 => if (value < 0) {
  NonTrnasferableERC20Votes.sol::3047 => if (len > 5) {
  NonTrnasferableERC20Votes.sol::3075 => if (pos == 0) {
  NonTrnasferableERC20Votes.sol::3104 => if (pos > 0) {
  NonTrnasferableERC20Votes.sol::3241 => if (len > 5) {
  NonTrnasferableERC20Votes.sol::3269 => if (pos == 0) {
  NonTrnasferableERC20Votes.sol::3298 => if (pos > 0) {
  NonTrnasferableERC20Votes.sol::3435 => if (len > 5) {
  NonTrnasferableERC20Votes.sol::3463 => if (pos == 0) {
  NonTrnasferableERC20Votes.sol::3492 => if (pos > 0) {
  NonTrnasferableERC20Votes.sol::3846 => if (from == address(0)) {
  NonTrnasferableERC20Votes.sol::3849 => if (to == address(0)) {
  NonTrnasferableERC20Votes.sol::3864 => if (from == address(0)) {
  NonTrnasferableERC20Votes.sol::3878 => if (to == address(0)) {
  NonTrnasferableERC20Votes.sol::3902 => if (account == address(0)) {
  NonTrnasferableERC20Votes.sol::3917 => if (account == address(0)) {
  NonTrnasferableERC20Votes.sol::3961 => if (owner == address(0)) {
  NonTrnasferableERC20Votes.sol::3964 => if (spender == address(0)) {
  NonTrnasferableERC20Votes.sol::4057 => if (initialOwner == address(0)) {
  NonTrnasferableERC20Votes.sol::4104 => if (newOwner == address(0)) {
  NonTrnasferableERC20Votes.sol::4222 => require($._hashedName == 0 && $._hashedVersion == 0, "EIP712: Uninitialized");
  NonTrnasferableERC20Votes.sol::4265 => if (bytes(name).length > 0) {
  NonTrnasferableERC20Votes.sol::4271 => if (hashedName != 0) {
  NonTrnasferableERC20Votes.sol::4287 => if (bytes(version).length > 0) {
  NonTrnasferableERC20Votes.sol::4293 => if (hashedVersion != 0) {
  NonTrnasferableERC20Votes.sol::4482 => if (from == address(0)) {
  NonTrnasferableERC20Votes.sol::4485 => if (to == address(0)) {
  NonTrnasferableERC20Votes.sol::4496 => if (from != to && amount > 0) {
  NonTrnasferableERC20Votes.sol::4497 => if (from != address(0)) {
  NonTrnasferableERC20Votes.sol::4505 => if (to != address(0)) {
  ERC20Upgradeable.sol::192 => if (from == address(0)) {
  NontransferableERC20Votes.sol::69 => require(msg.sender == address(manager), "Only manager can initialize");
  ERC20Upgradeable.sol::248 => if (account == address(0)) {
 
  
    

### Time spent:
20 hours