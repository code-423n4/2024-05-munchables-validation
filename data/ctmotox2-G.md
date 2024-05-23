**`G-01 Unused errors in LockManager.sol`**

In ILockManager.sol, 28 errors has been identified in order to be used in LockManager.sol

However, below 3 errors has not been used in the LockManager.sol

`OnlyAccountManagerError`
`USDPriceInvalidError`
`InvalidTokenContractError`

>**Recommendation**

Check if you need these errors or not. If yes, use the errors.

If not, delete the errors to save gas, clarify any confusion about them.