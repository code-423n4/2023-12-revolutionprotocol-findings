packages>revolution>src>builder>RevolutionBuilder.sol
function initialize on line 118: 

It seems reasonable to consider removing the redundant ownership check in the initialize function(line 120), given that a similar check already exists in the __Ownable_init_unchained function. The __Ownable_init_unchained function is called within the __Ownable_init(line 123) function, which, in turn, is called within the initialize function.