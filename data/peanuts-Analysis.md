### Summary of the protocol
- Protocol has a voting system and an auction system
- Protocol has ERC20Vote tokens and ERC721 tokens that have voting weight
- Users can buy ERC20Vote tokens anytime. Price of a token is according to the VRGDA formula
- To get ERC721 tokens, users have to bid in an auction
- For ERC721 tokens to be minted in an auction, the art piece must have the highest votes amongst all art pieces and reach a set quorum percentage
- Anyone can create an art piece.
- There is only one auction happening at one time

### Flow of the protocol
- A user creates an art piece and puts it into the protocol
- If a user likes the art piece, they will vote on it. (Votes can be bought (it is a ERC20 token, and it is not transferrable/burnable) )
- After some time, the piece with the highest votes will be selected. It will be created as an ERC721 token and auctioned off
- After the auction, the highest bidder will get the ERC721 NFT, and the creator of the art piece will get some ETH and ERC20 vote tokens.

### Approach

| Step | Approach                    | Details                                                                        | Comments                                                            |
| ---- | --------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| 1    | Reading the Documentation   | [Code4rena Page](https://code4rena.com/audits/2023-12-revolution-protocol#top) | No Gitbook / Documentation / Examples, difficult to understand      |
| 2    | Reading VRGDA               | [Paradigm](https://www.paradigm.xyz/2022/08/vrgda)                             | Decent understanding of protocol motives                            |
| 3    | Reading the forked versions | -                                                                              | Reading forked versions, such as zora auction house                 |
| 4    | Downloading the Code        | -                                                                              | Build is fine                                                       |
| 5    | Manual Code Review          | -                                                                              | Started with Auction & CultureIndex to get a top-down understanding |
| 6    | Manual Analysis             | -                                                                              | Started writing the Analysis report, and invariant testing          |
| 7    | RemiX                       | -                                                                              | Use RemiX to simulate heap creation                                 |
| 8    | Discord                     | -                                                                              | Read QnA to get a better understanding of the protocol              |
| 9    | Focused on Attack Ideas     | -                                                                              | Read through the 'attack ideas' section                             |

Overall Comments:

- Attack idea section is well-written, provided alot of starting points for auditing
- Needs proper documentation

### Codebase quality analysis and review

#### CultureIndex.sol

| Function              | Comments                                                                         | Analysis and Review                                                                   |
| --------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| validateMediaType     | Makes sure that certain media types do not have 0 value input                    | Not all MediaTypes are checked, such as AUDIO and OTHER, name and desc can be 0 value |
| validateCreatorsArray | Array cannot be larger than 100 creators, totalBPS must be equal to 10000 (100%) | 100 creators is still alot, may still cause OOG errors                                |
| createPiece           | validate array, mediatetype, insert into maxHeap, create a newPiece              | quorumVotes can be 0, block.number is used (might cause errors for L2)                |
| \_calculateVoteWeight | Adds the erc20Balance and erc721 balance                                         | Both must have same decimals, all erc721voting token has the same weight?             |
| \_getVotes            | Calculate the total votes of an account                                          | Call ERC20VotesUpgradeable.getVotes()                                                 |
| \_getPastVotes        | Check if the user has vote tokens during a certain block                         | Does it make it such that older votes are better than new votes?                      |

#### MaxHeap.sol

Testing in Solidity (Remove admin, upgradeability, constructor)

```
pragma solidity 0.8.0;

contract Heap {

     mapping(uint256 => uint256) public heap;

    uint256 public size = 0;

    /// @notice Mapping to keep track of the value of an item in the heap
    mapping(uint256 => uint256) public valueMapping;

    /// @notice Mapping to keep track of the position of an item in the heap
    mapping(uint256 => uint256) public positionMapping;

    /// @notice Get the parent index of a given position
    /// @param pos The position for which to find the parent
    /// @return The index of the parent node
    function parent(uint256 pos) private pure returns (uint256) {
        require(pos != 0, "Position should not be zero");
        return (pos - 1) / 2;
    }

    /// @notice Swap two nodes in the heap
    /// @param fpos The position of the first node
    /// @param spos The position of the second node
    function swap(uint256 fpos, uint256 spos) private {
        (heap[fpos], heap[spos]) = (heap[spos], heap[fpos]);
        (positionMapping[heap[fpos]], positionMapping[heap[spos]]) = (fpos, spos);
    }

    /// @notice Reheapify the heap starting at a given position
    /// @dev This ensures that the heap property is maintained
    /// @param pos The starting position for the heapify operation
    function maxHeapify(uint256 pos) internal {
        uint256 left = 2 * pos + 1;
        uint256 right = 2 * pos + 2;

        uint256 posValue = valueMapping[heap[pos]];
        uint256 leftValue = valueMapping[heap[left]];
        uint256 rightValue = valueMapping[heap[right]];

        if (pos >= (size / 2) && pos <= size) return;

        if (posValue < leftValue || posValue < rightValue) {
            if (leftValue > rightValue) {
                swap(pos, left);
                maxHeapify(left);
            } else {
                swap(pos, right);
                maxHeapify(right);
            }
        }
    }

    /// @notice Insert an element into the heap
    /// @dev The function will revert if the heap is full
    /// @param itemId The item ID to insert
    /// @param value The value to insert
    function insert(uint256 itemId, uint256 value) public {
        heap[size] = itemId;
        valueMapping[itemId] = value; // Update the value mapping
        positionMapping[itemId] = size; // Update the position mapping

        uint256 current = size;
        while (current != 0 && valueMapping[heap[current]] > valueMapping[heap[parent(current)]]) {
            swap(current, parent(current));
            current = parent(current);
        }
        size++;
    }

    /// @notice Update the value of an existing item in the heap
    /// @param itemId The item ID whose vote count needs to be updated
    /// @param newValue The new value for the item
    /// @dev This function adjusts the heap to maintain the max-heap property after updating the vote count
    function updateValue(uint256 itemId, uint256 newValue) public {
        uint256 position = positionMapping[itemId];
        uint256 oldValue = valueMapping[itemId];

        // Update the value in the valueMapping
        valueMapping[itemId] = newValue;

        // Decide whether to perform upwards or downwards heapify
        if (newValue > oldValue) {
            // Upwards heapify
            while (position != 0 && valueMapping[heap[position]] > valueMapping[heap[parent(position)]]) {
                swap(position, parent(position));
                position = parent(position);
            }
        } else if (newValue < oldValue) maxHeapify(position); // Downwards heapify
    }

    /// @notice Extract the maximum element from the heap
    /// @dev The function will revert if the heap is empty
    /// @return The maximum element from the heap
    function extractMax() external returns (uint256, uint256) {
        require(size > 0, "Heap is empty");

        uint256 popped = heap[0];
        heap[0] = heap[--size];
        maxHeapify(0);

        return (popped, valueMapping[popped]);
    }

    /// @notice Get the maximum element from the heap
    /// @dev The function will revert if the heap is empty
    /// @return The maximum element from the heap
    function getMax() public view returns (uint256, uint256) {
        require(size > 0, "Heap is empty");
        return (heap[0], valueMapping[heap[0]]);
    }


}
```

1. Deploy the contract and call insert with (1,0), (2,0) and (3,0) -> 3 art pieces created
2. Call updateValues with (1,1e18), (2,2e18), (3,3e18) -> 1st art piece gets 1 vote, 2nd art piece gets 2 vote, 3rd art piece gets 3 votes
3. Check the necessary functions

- heap (0)/(1)/(2) returns (3)/(1)/(2) (1 and 2 doesn't change place, probably because of the heap structure)

4. Call extractMax()

- heap (0)/(1) returns (2)/(1) (Seems like extractMax() fixes things)

5. Call insert with (4,0) and updateValues with (4,10e18) -> 4th art piece created, 10 votes on the piece

- getMax() returns (4,10e18) (This is correct)
- heap (0)/(1)/(2) returns (4)/(1)/(2) (The fourth art piece becomes the top heap, which is correct)

6. Call extractMax() and getMax()

- heap (0)/(1) returns (2)/(1) (2 becomes the top heap now, which is correct)
- getMax() returns (2,2e18)

- extractMax() works
- insert() works
- updateValues() works
- getMax works

Only issue would be the centralization issue, Admin can directly call `updateValue()` to manipulate the vote count.

### Mechanism Review

##### Creating Art Pieces in CultureIndex.sol

- Important checks are in place, like validating creator array. Never fully validate as creator can add the same address, resulting in griefing during auction
- MediaTypes is not validated fully. Name and description can be blank or repeated. Unsure if intended
- MaxHeap is correctly inserted
- quorumVotes can be quite scary, if totalVotesSupply is 0 or an extremely huge number. If 0, then no need votes to pass. If too big of a number, then might need a lot of votes
- Max limit of 100 creators can be gas intensive, griefing or not.

##### Dropping Top Voted Piece in CultureIndex.sol

- Called when ERC721 token is being created in auction
- mint process must be less than 750,000 gas, this amount should be flexible
- Top voted piece is being selected. This conflicts with quorum percentage. A piece with 100% percentage will lose to another piece with 60% quorum percentage because it has less votes, but since votes only can affect future pieces, the newer pieces will always have more votes. Quorum percentage is not useful in this sense
- ExtractMax will update the heap.
- Once a piece is dropped, cannot vote anymore. A piece that is dropped but have no auction will not be placed back into the heap

##### Auction Process in AuctionHouse.sol

- Whole contract starts with a pause state.
- Auction can only happen when protocol is unpaused
- When protocol is paused, it doesn't affect the current auction. Users still can createBid but not settle auction
- Only one auction at a time.
- When creating bid, must have a bid increment. Also, the highest bidder will have to refund the previous bidder, can get caught in a 50,000 gas trap, but not permanent DoS
- There is a time extension if createbid is called moments before auction ends. Potential to lengthen auction but don't see the reason. Extension duration will also be pretty short
- When settling auction, if no bid, the nft is burned.
- transferFrom is used instead of safeTransferFrom, which is good to prevent callback attacks, but doesn't check if highest bidder is a contract. If contract, will lose the NFT
- If entropyrate is not 10,000, erc20TokenEmitter.buyToken will be called
- Auction winner gets NFT, owner gets a cut of the ETH, creator(s) gets a cut of the ETH.

##### Buying an ERC20Token in ERC20TokenEmitter.sol

- Can buy for a list of addresses, can buy only for 1 address as well
- builder,deployer,purchasereferral gets a small sum of ETH
- creator gets a sum as well (unsure who the creator is in this case, if a person directly calls buyToken). Creators get their payment through ERC20Vote tokens
- Token price is quoted using the VRGDA calculation
- total token gained is split amongst the addresses

### Centralization risks

| Contract          | Function                         | Risk   | Explanation                                                                                                  |
| ----------------- | -------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------ |
| CultureIndex      | SetQuorumVotesBPS                | Medium | Able to control the quorum threshold, only can set until 9999                                                |
| CultureIndex      | AuthorizeUpgrade                 | None   | Blacklists a user, cannot transfer stUSDe                                                                    |
| AuctionHouse      | settleCurrentAndCreateNewAuction | High   | Controls an integral part of the protocol, which is auctioning                                               |
| AuctionHouse      | setCreatorRateBps                | High   | Sets how much goes to the creator and how much goes to the auction owner                                     |
| AuctionHouse      | setEntropyRateBps                | Low    | Sets how much ETH goes to the creators and how much ERC20 tokens goes to the creators                        |
| AuctionHouse      | setReservePrice                  | Low    | Sets the lowest price an auction participant can bid                                                         |
| AuctionHouse      | setMinBidIncrementPercentage     | High   | Can grief the auction, also no upper limit                                                                   |
| AuctionHouse      | setTimeBuffer                    | High   | No limit, can set an extraordinary limit to grief auction                                                    |
| ERC20TokenEmitter | setCreatorRateBps                | High   | Sets how much goes to the creator and how much goes to the auction owner                                     |
| ERC20TokenEmitter | setEntropyRateBps                | Low    | Set how much goes to the creators and how much goes to mint ERC20 tokens                                     |
| VerbsToken        | mint                             | High   | Minter can maliciously inflate the votes count by bypassing the auction and minting all available art pieces |

- Overall Centralization risks: High
- Owner controls the payment, integral parts of the protocol, and the quorum votes

A way to lessen the centralization risks is to set one constant value, such as setting the minBidIncrementPercentage once. However, this cannot be the practice for everything, as the protocol needs to balance the protocol base on real world sentiment, like if the protocol sets the entropy rate too low, some people might not like it.

Some function controls are pretty risky if an owner turns malicious, such as setting the time buffer to 100 years, or setting the minBidIncrement to 1000e20. Best to have a multi-sig owner.

### Systemic Risks

- The total supply of vote token is ever increasing, which leaves a lot of votes being unused in the future (users with vote tokens not participating in the protocol actively anymore). This will affect the quorum percentage because art pieces might not reach the quorum percentage
- The auction can be stalled and griefed by setting 100 creators with the same address (gas grief)
- Only ETH is accepted, which may affect the participation rate of the protocol
- Auction may not be that lucrative since all ERC721 tokens have the same vote weight

### Time spent:
030 hours