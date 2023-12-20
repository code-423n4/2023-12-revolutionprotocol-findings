## [N-01] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword   

There is 1 instance of this issue in 1 file:

    File: CultureIndex.sol	

    69: mapping(uint256 => mapping(address => Vote)) public votes;

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol

## [N-02] Consider addings checks for signature malleability

Use OpenZeppelin’s *ECDSA* contract rather than calling *ecrecover()* directly

There is 1 instance of this issue in 1 file:

    File: CultureIndex.sol	

    435: address recoveredAddress = ecrecover(digest, v, r, s);

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/CultureIndex.sol

## [N-03] Event is missing *indexed* fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (threefields). Each *event* should use three *indexed* fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

There are 3 instances of this issue in 1 file:

    File: NontransferableERC20Votes.sol	

    95: function transfer(address, uint256) public virtual override returns (bool) {

    109: function transferFrom(address, address, uint256) public virtual override returns (bool) {

    116: function approve(address, uint256) public virtual override returns (bool) {

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/main/packages/revolution/src/NontransferableERC20Votes.sol