## [L-01] ` CultureIndex.sol::dropperAdmin` cannot be changed once it is set in `initiliaze` function

There is no setter function for admin users to reset `dropperAdmin` address in `CultureIndex.sol`. In case this user acts maliciously or compromised, there is no way to reset  `dropperAdmin` address from the contract. 


### Recommendations

Consider implement 2step address setter function to set `dropperAdmin` address in `CultureIndex.sol`.
