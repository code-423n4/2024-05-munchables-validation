## [QA-1] Lack of Validation for Proposed Token Contracts in proposeUSDPrice

## Proof of Concept

```
    function proposeUSDPrice(
        uint256 _price,
        address[] calldata _contracts
    )
```
The contract's proposal process for updating the USD price lacks validation to ensure that all proposed token contracts have been previously configured within the system. There is no check within this function to verify if the proposed token contracts are already included in the list of supported tokens (configured tokens). This creates a situation where  actors could propose updates for token addresses that will eventually fail during execution once the desired threshold is met. The creation of the proposal itself should not be possible if the contracts used have not been configured.

Add a  validation  involving  checking each proposed token contract against the `configuredTokens` mapping to verify its configuration status before allowing it to be included in proposals.

## [QA-2] Incorrect Handling of Tokens with More Than 18 Decimals in getLockedWeightedValue

The `getLockedWeightedValue` function in the `LockManager` contract incorrectly assumes that all ERC20 tokens have a maximum of 18 decimals.

The code ` deltaDecimal = 10 ** (18 - configuredTokens[configuredTokenContracts[i]].decimals) calculates the difference between 18 (assumed max decimals) and the actual token decimals.` will always revert if the protocal team decides to add a token with more than 18 decimal places.

The calculation should be modified to correctly handle tokens with more than 18 decimals. 


