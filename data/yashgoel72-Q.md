### [Low] User Can Disapprove and Then Approve a Proposal

### Description
The `approveUSDPrice` function allows a user with a role in `PriceFeed` to approve a USD price update proposal. Similarly, the `disapproveUSDPrice` function allows the user to disapprove a USD price update proposal. However, the current implementation of approveUSDPrice lacks a check to prevent a user who has already disapproved a proposal from later approving it. This oversight can be exploited by a user to first disapprove and then approve the same proposal, potentially manipulating the proposal outcome.
[function approveUSDPrice()](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L177)

### Recommended Mitigation
To address this issue, add a check in the `approveUSDPrice` function to revert the transaction if the user has already disapproved the proposal.
```diff
       if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)
           revert ProposalAlreadyDisapprovedError();  // New check added
```