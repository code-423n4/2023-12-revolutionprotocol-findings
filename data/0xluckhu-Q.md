In ```ERC20TokenEmitter::buyToken```, it mints tokens to addresses per the ```basisPointSplits``` (line 210-213). Because of **rounding down** in the calculation of the mint amount to each address, the total minted amount may be smaller than the ```totalTokensForBuyers```. And in particular, if the ```totalTokensForBuyers``` and each ```basisPointSplits[i]``` is not big enough, all addresses may get 0 token minted.

    if (totalTokensForBuyers > 0) {
       // transfer tokens to address
       _mint(addresses[i], uint256((totalTokensForBuyers * int(basisPointSplits[i])) / 10_000));
    }

To protect users funds, it's recommend to require the ```basisPointSplits[i]``` shall be bigger than some MIN value, also the ```totalTokensForBuyers``` shall be bigger than some MIN value.