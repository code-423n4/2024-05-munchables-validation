The `approveUSDPrice` function approves any proposed proposal. However, it does not verify if the proposal has already been disapproved. So the `Price_Feed` role can accidently approve a disapproved proposal and accidently up a proposal.

## Mitigation
Please add this in `approveUSDPrice` function.
```Diff
+ if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)
+           revert ProposalAlreadyDisapprovedError();
```