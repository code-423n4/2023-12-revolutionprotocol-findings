1. packages>revolution>src>builder>RevolutionBuilder.sol
function initialize on line 118: 

It seems reasonable to consider removing the redundant ownership check in the initialize function(line 120), given that a similar check already exists in the __Ownable_init_unchained function. The __Ownable_init_unchained function is called within the __Ownable_init(line 123) function, which, in turn, is called within the initialize function.

2. packages>revolution>src>Descriptor.sol

function initialize on line 78:
1. We ensure the caller is the contract manager 2 consecutive times(line 79 & line 82)
2. We ensure _initialOwner is not address(0) 2 times: on line 84 and line 87: __Ownable_init --> __Ownable_init_unchained

3. packages>revolution>src>CultureIndex.sol
function createPiece line:236
So basically we can spare the 2nd for loop that emits the PieceCreatorAdded event and we can emit it in the first for loop as it doesn't rely on newPiece.