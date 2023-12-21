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

Overall the codebase is organised well. Each of the core contracts for the protocol are well defined and the functionality is divided well into the relevant contracts. As well as this the contents of each function a clear and easy to follow. One thing to note is that the layout of the functions within each contract doesn't always have a clear reasoning and would be best to follow best practices from the solidity documentation (ordered in terms of visibility from external -> private. The code is relatively well commented but can be patchy in places whilst being more verbose in others.

The project has a sophisticated test suite set up in foundry, and it's relatively straightforward for outsiders to patch in their own tests as they see necessary. However it would be advisable to add more exhaustive tests, particularly focused on potential edge cases to cover all possible bases.

The codebase appears to have the relevant events being emitted on key functions. One thing to note is that there appears to be a mix of require statements/custom errors and reverts with error strings used across the codebase. It would be best practices where possible to stick to one style for the contracts checks to improve readability.

There are also a number of smaller tweaks that would be possible to help improve the gas usage of the contracts within the project. These include but are not limited to things such as caching array lengths before looping over them, using custom errors,

## Centrality Risk

The `owner` of the contracts in the protocol has permissions related to contract upgrades, pausing/unpausing the protocol as well as setting values of various things across the projects core contracts (functions highighted in blue on the miro board).

This is something to remain keenly aware of as it means there's a large degree of responsibility in the hands of the owner, whilst also being a large amount of admin to decide over if controlled by a DAO. 


### Time spent:
12 hours