# Revolution Protocol Report

The following Miro board was used to get a high level overview of the functionality of the Revolution protocol, as well as point out all the likely entry points at a would-be attackers disposal.

[Revolution Miroboard](https://i.imgur.com/CkAuN8k.png)

## User endpoints
5 main user actions were highlighted as areas likely to provide entry for malicious actors, these are as follows:

- Creating new art pieces
- Voting on existing art pieces
- Settling/Creating new auctions
- Bidding on existing auctions
- Purchasing the untradeable ERC20 associated with the protocol.

## Potential for protocol error
On top of this a few areas where it's most likely the protocol could have made mistakes were highlighted as well, these are as follows:

- Their handling of upgradeable contracts
- Possible arithmetic errors when calculating prices/reward amounts

## Codebase Assessment

Overall the codebase is organised very well. Each of the core contracts for the protocol are well defined and the functionality is divided well into the relevant contracts.

The project has a sophisticated test suite set up in foundry, and it's relatively straightforward for outsiders to patch in their own tests as they see necessary. However it would be advisable to add more extensive tests, particularly focused on potential edge cases to cover all possible bases.

## Centrality Risk

The `owner` in the protocol has a lot of power over the protocol and 

### Time spent:
12 hours