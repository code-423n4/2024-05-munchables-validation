### Redundant Code in USD Price Proposal

The `proposeUSDPrice` function contains a line to delete the current `usdUpdateProposal` which might be redundant and
leads to unnecessary gas consumption.

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L161

This `delete` statement is redundant and wastes gas on operations that might not be needed could increase transaction
costs unnecessarily.

Recommend removing the redundant codes to save gas.

### Redundant Check in `_execUSDPriceUpdate`

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L507C1-L510C9

```solidity
    function _execUSDPriceUpdate() internal {
        if (
            usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD &&
            usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD
        ) {
            uint256 updateTokensLength = usdUpdateProposal.contracts.length;
            for (uint256 i; i < updateTokensLength; i++) {
                address tokenContract = usdUpdateProposal.contracts[i];
                if (configuredTokens[tokenContract].nftCost != 0) {
                    configuredTokens[tokenContract].usdPrice = usdUpdateProposal
                        .proposedPrice;

                    emit USDPriceUpdated(
                        tokenContract,
                        usdUpdateProposal.proposedPrice
                    );
                }
            }

            delete usdUpdateProposal;
        }
    }
```

The `if` check is redundant and can be removed to save gas. 

1. By the time `_execUSDPriceUpdate` is called, it is typically expected (or should be ensured by the calling function) that the approval count condition (`usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD`) has already been met. This is because the function would only be triggered in a context where the proposal has already achieved sufficient approvals.


2. If `usdUpdateProposal.disapprovalsCount >= DISAPPROVE_THRESHOLD` meets, the proposal will be deleted. 

Recommend removing the `if` check in the `_execUSDPriceUpdate` function to save gas. 