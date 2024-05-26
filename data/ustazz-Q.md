### [Non Critical - 1] check for disapproval given is missing in LockManager.sol::approveUSDPrice(uint256) 

***Description:***

In `LockManager.sol::disapproveUSDPrice(uint256)`, there are two checks as following:

```solidity
225        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)
226            revert ProposalAlreadyApprovedError();
227        if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)
228            revert ProposalAlreadyDisapprovedError();
```

However, in `LockManager.sol::approveUSDPrice(uint256)`, only one condition is checked:

```solidity
194        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)
195            revert ProposalAlreadyApprovedError();
```

A check should be added in `LockManager.sol::approveUSDPrice(uint256)` to replicate the logic for both cases.