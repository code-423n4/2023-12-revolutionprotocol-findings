# QA-01 creatorAddress low level call can block settling of an auction

## Impact

Protocol uses low level call to transfer native amounts but one of the address used has no defined contract (in the current provided source code) even in tests and can block the completion of settling an auction.

## Proof of Concept

When an auction want to settle it also calls `ERC20TokenEmitter#buyToken`. Inside this contract when native amount are transferred the low level call is used. More exactly there is the [treasury address](https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L191) 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L191

which is defined as the `VerbsDaoLogicV1`. `VerbsDaoLogicV1` has the *receive* method defined so can receive native transfers.

But there is also the *creatorAddress* address which is not defined as specific contract and in tests is used just as a address (lika EOA which can receive native transfers) but in this case can be also an contract.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L196



## Recommendation

Instead of using low level `call` make "globally" available the function `_safeTransferETHWithFallback` when native transfers are made




# QA-02 MaxHeap#insert does not check if the itemId already exists

## Impact

In case of an upgrade. If the code ever calls the `MaxHeap#insert` function with the same itemId it can cause problems because this would need to be called as `MaxHeap#updateValue` instead

## Proof of Concept

The insert methods as it is does not verify if the itemId already exists 

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L119-L130

it presumes as the itemId being added is a new one.


## Recommendation

Before insert check if itemId is not already in the heap

```diff
    function insert(uint256 itemId, uint256 value) public onlyAdmin {
+       require(valueMapping[itemId] == 0 && positionMapping[itemId] == 0, "Item id already in heap");
+
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
```



# QA-03 MaxHeap has not defined an maximal possible count of elements it can contain which can cause problems when extracting element from the heap

## Impact

Operation for extracting the maximal value from the `MaxHeap` contract in the worst case is log₂("number of elements in heap") but the maximal size of elements has the upper bound of max(uint256) which can become problematic.

## Proof of Concept

The comment for `MaxHeap#insert` states that:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L116

But this revert happens when

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/MaxHeap.sol#L129

`size` overflows which will happen on max(uint256) + 1

So there can be a lot of elements in the heap.

The `extractMax()` when taking/popping the max element does also a *downwards heapify operation* which is recursive operation and in the worst case is a log₂("nubmer of elements in heap") and can cause ouf of gas errors. 


## Recommendation

Define an max value of possible elements for the heap/`MaxHeap` that is not to big to cause problems.




# QA-04 owner of AuctionHouse should not be allowed to change parameters for the ongoing auction

## Impact

The auction house owner can change many parameters that can influence the outcome of the ongoing auction.

## Proof of Concept

The following parameters are globally defined and can cause problems on the ongoing auction:

* the reserve price

should not be changed during the auction. This can influence users from participating in biding on ongoing auction.

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L179

* min bid increment percentage

Have a similar effect as the reserve price

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L180-L183

* https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L277

This parameter can influence the extending of an almost close auction.


## Recommendation

All the mentioned parameters should stay fixed during the lifetime of the current auction. So the users that interacts with the auction can have a clearer view of whats going on.

When the auction is created (the `Auction` struct) it could contain a copy of this globally defined values and use those instead.




# QA-05 AuctionHouse duration has no minimal/maximal value check


## Impact

When an AuctionHouse is created/initialized it does not check the minimum/maximum validity of an auction duration. Which can cause problems later when the auction is started.


## Proof of Concept

In the constructor there is no verification

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L137

and when the auction is started it just uses whatever was passed in the initialization time

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L315


## Recommendation

Have a require in the `initialize` function to limit the minimal and maximal `duration` parameter of an auction.




# QA-06 CultureIndex MediaType does not validate the audio type art piece

## Impact

The protocol has defined `MediaType.AUDIO` type but has no validation in place for this type not has a field that represents this type.

## Proof of Concept

The `CultureIndex.MediaType` has the following types available:

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L77-L85

the `validateMediaType` function misses the `AUDIO` type validation

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L159-L168

Also the `ArtPieceMetadata` structure itself doesnt have a parameter that would be used for the `AUDIO` type

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/interfaces/ICultureIndex.sol#L87-L95

## Recommendation

Change the `ArtPieceMetadata` to contain an unified url for types like AUDIO IMAGE ANIMATION

```diff
    // Struct defining metadata for an art piece.
    struct ArtPieceMetadata {
        string name;
        string description;
        MediaType mediaType;
-       string image;
        string text;
-       string animationUrl;
+       string url; // used for things like image, animationUrl and audioUrl
    }
```



# QA-07 documentation discrepancy for MediaType.ANIMATION in CultureIndex#createPiece

## Impact

The documentation for `createPiece` function is ambiguous for the MediaType animation.

## Proof of Concept

For the ANIMATION type in `createPiece` function it says that the Animation URL is optional

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L205


but if we check the `validateMediaType` function which checks different media types we can see that if we miss the animation url it will revert the transaction

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L164-L165


## Recommendation

Rewrite code or documentation to match the described functionality.



# QA-08 CultureIndex#createPiece metadata parameter can be reused many times

## Impact

Protocol will have many copies of art pieces where users will have a hard time to choose which of the many copies to choose for voting.

## Proof of Concept

Inside the `createPiece` function there is only validation for the creatorArray parameter and for correctness of the mediatype.
But there is no prevention to reuse this the parameters many times creating a lot of copies of the same art piece.


## Recommendation

One solution would be to add a mapping of the mediatype ipfs urls. In case a url is known throw the transaction

For example:

```diff
    function createPiece(
        ArtPieceMetadata calldata metadata,
        CreatorBps[] calldata creatorArray
    ) public returns (uint256) {
        uint256 creatorArrayLength = validateCreatorsArray(creatorArray);


        // Validate the media type and associated data
        validateMediaType(metadata);

        uint256 pieceId = _currentPieceId++;
+       
+       string url;
+       if (metadata.mediaType == MediaType.TEXT) {
+           bytes32 textHash = keccak256(abi.encodePacked(metadata.text));
+           // Convert bytes32 to string
+           bytes memory bytesData = new bytes(32);
+           assembly {
+               mstore(add(bytesData, 32), textHash)
+           }
+           
+           url = string(bytesData);
+       }
+       else {
+           // Note : in reality the ArtPieceMetadata structure has many `different` urls ..but I think this can be condensed to only one 
+           url = metadata.url;
+       }
+       revert(knownUrls[url] == 0, "Art piece already exists")
+       knownUrls[url] = pieceId; // or 1 or true... 
 
        ...
```


