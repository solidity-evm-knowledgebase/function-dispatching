# function dispatching

Function dispatch is the algorithm used to determine which functions/commands should be run in response to a message.

This message is the calldata, which tells the smart contract what to do. The first four bytes of the calldata is the function selector which will be used to determine which function should be called.

Solidity handles this for us. In Huff it would look something like that:

We can get the function selector by doing:

```Huff
0x00 calldataload 0xE0 shr
````

This loads the four byte function selector onto the stack. 0x00 calldataload will load 32 bytes starting from position 0 onto the stack (if the calldata is less than 32 bytes then it will be right padded with zeros). 0xE0 shr right shifts the calldata by 224 bits, leaving 32 bits or 4 bytes remaining on the stack.

We can then proceed to check if this function selector matches any of our defined functions. If yes, we can jump to the respective function, and continue with our logic.

From the Huff documentation, an example of what a dispatcher would look like for a standard ERC-20 token:

```Huff
// Interface
#define function allowance(address,address) view returns (uint256)
#define function approve(address,uint256) nonpayable returns ()
#define function balanceOf(address) view returns (uint256)
#define function DOMAIN_SEPARATOR() view returns (bytes32)
#define function nonces(address) view returns (uint256)
#define function permit(address,address,uint256,uint256,uint8,bytes32,bytes32) nonpayable returns ()
#define function totalSupply() view returns (uint256)
#define function transfer(address,uint256) nonpayable returns ()
#define function transferFrom(address,address,uint256) nonpayable returns ()
#define function decimals() nonpayable returns (uint256)
#define function name() nonpayable returns (string)
#define function symbol() nonpayable returns (string)

// Function Dispatching
#define macro MAIN() = takes (1) returns (1) {
    // Identify which function is being called.
    0x00 calldataload 0xE0 shr          // [func_sig]

    dup1 __FUNC_SIG(permit)             eq permitJump           jumpi
    dup1 __FUNC_SIG(nonces)             eq noncesJump           jumpi

    dup1 __FUNC_SIG(name)               eq nameJump             jumpi
    dup1 __FUNC_SIG(symbol)             eq symbolJump           jumpi
    dup1 __FUNC_SIG(decimals)           eq decimalsJump         jumpi
    dup1 __FUNC_SIG(DOMAIN_SEPARATOR)   eq domainSeparatorJump  jumpi

    dup1 __FUNC_SIG(totalSupply)        eq totalSupplyJump      jumpi
    dup1 __FUNC_SIG(balanceOf)          eq balanceOfJump        jumpi
    dup1 __FUNC_SIG(allowance)          eq allowanceJump        jumpi

    dup1 __FUNC_SIG(transfer)           eq transferJump         jumpi
    dup1 __FUNC_SIG(transferFrom)       eq transferFromJump     jumpi
    dup1 __FUNC_SIG(approve)            eq approveJump          jumpi

    // Revert if no match is found.
    0x00 dup1 revert

    allowanceJump:
        ALLOWANCE()
    approveJump:
        APPROVE()
    balanceOfJump:
        BALANCE_OF()
    decimalsJump:
        DECIMALS()
    domainSeparatorJump:
        DOMAIN_SEPARATOR()
    nameJump:
        NAME()
    noncesJump:
        NONCES()
    permitJump:
        PERMIT()
    symbolJump:
        SYMBOL()
    totalSupplyJump:
        TOTAL_SUPPLY()
    transferFromJump:
        TRANSFER_FROM()
    transferJump:
        TRANSFER()
}
```
