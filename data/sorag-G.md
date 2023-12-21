### Don't use variable in VerbsToken/_mintTo function
If you don't use artPiece variable  in VerbsToken/_mintTo and  derectly use _artPrice, you can save gas approximetely 8 Gas.
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/VerbsToken.sol#L292-L308

