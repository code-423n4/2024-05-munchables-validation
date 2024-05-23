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


### 2 setUSDThresholds()
The APPROVE_THRESHOLD & DISAPPROVE_THRESHOLD are not validated and hence could be any number. If these two are set high, example 25 each and a proposal is initiated, then until that threshold is met, the whole proposal/approval mechanism will be stuck. Even the admin will not be able to override the values until the proposal is either approved or disapproved.

Recommendation is to restrict the threshold values to a viable max value to avoid the possibility of that state.
