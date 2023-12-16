In ```ERC20TokenEmitter::setCreatorsAddress```, the ```owner``` can update the ```creatorsAddress```. However, there's no need to use the ```nonReentrant``` guard here. Suggest to remove it.

    function setCreatorsAddress(address _creatorsAddress) external override onlyOwner nonReentrant {
        require(_creatorsAddress != address(0), "Invalid address");

        emit CreatorsAddressUpdated(creatorsAddress = _creatorsAddress);
    }