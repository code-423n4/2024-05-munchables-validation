### Redundant Code in USD Price Proposal

The `proposeUSDPrice` function contains a line to delete the current `usdUpdateProposal` which might be redundant and
leads to unnecessary gas consumption.

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L161

This `delete` statement is redundant and wastes gas on operations that might not be needed could increase transaction
costs unnecessarily.

Recommend removing the redundant codes to save gas.