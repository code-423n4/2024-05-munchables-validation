### [L-01] If APPROVE_THRESHOLD is greater then 5 or DISAPPROVE_THRESHOLD proposal will stuck

##### Description: 
Only five roles can approve or disapprove the proposal. But there is no check for the maximum number of APPROVE_THRESHOLD or DISAPPROVE_THRESHOLD which can cause a proposal to get stuck.


##### Proof of Concept:

let's build a scenario

suppose usdUpdateProposal is empty.
1. Admin changes APPROVE_THRESHOLD or DISAPPROVE_THRESHOLD or both more than 5.
2. One of Role.pricefeed purposes usdPrice proposal.
3. Suppose all Role.pricefeed approve the proposal but because APPROVE_THRESHOLD is greater than 5, the proposal won't get approved.

that's how the proposal gets stuck.

##### Recommended Mitigation:
**link to code:**https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L129C1-L139C6

```diff
    function setUSDThresholds(
        uint8 _approve,
        uint8 _disapprove
    ) external onlyAdmin {
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
+       if(_approve > 5 || _disapprove > 5)
+           revert();
        APPROVE_THRESHOLD = _approve;
        DISAPPROVE_THRESHOLD = _disapprove;

        emit USDThresholdUpdated(_approve, _disapprove);
    }
```

### [N-01] Deleting struct usdUpdateProposal is an extra step

##### Description: 
While proposing a new proposal, a struct named "usdUpdateProposal" is already empty and there is no need to delete it.

##### Instance:
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L161

### [N-02] Modifier's logic is not following NATspec

##### Description: 
NATspec says that "Token supplied must be configured and active" but the modifier is only checking whether is supplied token is active or not

##### Instance:
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L53C1-L58C6