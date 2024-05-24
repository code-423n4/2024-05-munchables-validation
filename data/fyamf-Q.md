## Issue1
When approving USD price, it is not checked that the `msg.sender` has already disapproved it or not. It only checks that it was not approved before by `msg.sender`.

```solidity
    function approveUSDPrice(
        uint256 _price
    )
        external
        onlyOneOfRoles(
            [
                Role.PriceFeed_1,
                Role.PriceFeed_2,
                Role.PriceFeed_3,
                Role.PriceFeed_4,
                Role.PriceFeed_5
            ]
        )
    {
        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();
        if (usdUpdateProposal.proposer == msg.sender)
            revert ProposerCannotApproveError();
        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)
            revert ProposalAlreadyApprovedError();

        //.....
    }
```
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L177

So, it is possible that a price feed role approves and disapproves a proposal. First, he disapproves, and then he approves. 

Solution is to modify the code as follows:
```
function approveUSDPrice(
        uint256 _price
    )
        external
        onlyOneOfRoles(
            [
                Role.PriceFeed_1,
                Role.PriceFeed_2,
                Role.PriceFeed_3,
                Role.PriceFeed_4,
                Role.PriceFeed_5
            ]
        )
    {
        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();
        if (usdUpdateProposal.proposer == msg.sender)
            revert ProposerCannotApproveError();
        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)
            revert ProposalAlreadyApprovedError();

        // add this line
        if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)
            revert ProposalAlreadyDisapprovedError();

        //.....
    }
```