# Unsupported Opcode in Multi-Chain Deployment

The pragma used `pragma solidity ^0.8.22;` isn't yet supported by Base and Optimism as stated in (Base docs)[https://docs.base.org/differences/]: 

>Aside from the above, Base Mainnet does not yet support the new PUSH0 opcode >introduced in Shanghai, which is the default target for the Solidity compiler if >you use version 0.8.20 or later.
>
>We recommend using 0.8.19 or lower until this is upgraded.

As recommended by Base docs, use 0.8.19 to avoid incompatibility.