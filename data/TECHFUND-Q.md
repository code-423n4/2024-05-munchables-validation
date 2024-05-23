### 1) appoveUSDPrice()
The disapprover of the proposer should not be able to approve the USD Price Proposal.

The check to ensure disapprover is also not allowed is missing in the validation.

```
       if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();
        if (usdUpdateProposal.proposer == msg.sender)
            revert ProposerCannotApproveError();
        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)
            revert ProposalAlreadyApprovedError();
        if (usdUpdateProposal.proposedPrice != _price)
            revert ProposalPriceNotMatchedError();
```

add the below condition to the approve function before approving.

       if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)
            revert ProposalAlreadyDisapprovedError();