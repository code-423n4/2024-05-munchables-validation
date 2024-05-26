`approveUSDPrice` should check whether `msg.sender` already disapproved the USD price proposal, and raise `ProposalAlreadyDisapprovedError` if it does. Otherwise, the proposer may simultaneously *approve* and *disapprove* the proposal, which does not make much sense.