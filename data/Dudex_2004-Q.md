The `approveUSDPrice` function can approve any proposed proposal. However, it does not verify if the proposal has already been disapproved. So the `Price_Feed` role can accidently approve a disapproved proposal causing a down approval into up.

## Mitigation
Please add this check in `approveUSDPrice` function.
```Diff
+ if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)
+           revert ProposalAlreadyDisapprovedError();
```