# [L-1] Tautology in `LockManager.sol#_execUSDPriceUpdate`
```
if (
    usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD &&
    usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD
)
```
`usdUpdateProposal.disapprovalsCount` is updated in `LockManager.sol#disapproveUSDPrice`
```
if (usdUpdateProposal.disapprovalsCount >= DISAPPROVE_THRESHOLD) {
    delete usdUpdateProposal;

    emit RemovedUSDProposal();
}
```
If `disapprovalsCount` is equal to the threshold, it then proceeds to delete the proposal.  
Therefore, `usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD` would always return true when calling `_execUSDPriceUpdate`  

# [L-2] Proposer can only propose one price which it conflicts the ability to propose many token contracts at once
Proposers can propose to update price for many tokens at once.
[LockManager.sol#L142-L145](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L142-L145)
```
 function proposeUSDPrice(
    uint256 _price,
    address[] calldata _contracts
)
```
However, they can only propose one price data, and if the proposal is approved then all tokens would be assigned the same price.  
This doesn't apply to the real world scenario where the occurrence of different tokens having the same price is unlikely. 

## Recommendation  
- Either allow proposers to propose only one price for one token or allow multiple prices for each token to be submitted.  

# [L-3] LockManager doesn't support tokens with high decimals (> 18)  
In `LockManager.sol#getLockedWeightedValue`, the normalization of the token value only support token with <= 18 decimals
[LockManager.sol#L472-L482](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L472-L482)
```
// We are assuming all tokens have a maximum of 18 decimals and that USD Price is denoted in 1e18
uint256 deltaDecimal = 10 **
    (18 -
        configuredTokens[configuredTokenContracts[i]].decimals);
lockedWeighted +=
    (deltaDecimal *
        lockedTokens[_player][configuredTokenContracts[i]]
            .quantity *
        configuredTokens[configuredTokenContracts[i]]
            .usdPrice) /
    1e18;
```
This conflicts with the expected behavior of ERC20 token in the source of truth where it clearly states to support token with high decimals.