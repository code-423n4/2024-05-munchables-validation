## Q/A-01 : `PlayerSettings` struct contains only one element; `lockduration` . Can be used directly to reduce code complexity .
`struct PlayerSettings ` has only one element which is `lockDuration`  . Instead of using the struct , lockDuration can be used directly to decrease the code complexity . 
```solidity 
 struct PlayerSettings {
        uint32 lockDuration;
    }

 mapping(address => PlayerSettings) playerSettings;   
```
code link : https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L26

## Q/A-02 : Not enough input validation in `setUSDThresholds` . 
In `setUSDThresholds` function , `APPROVE_THRESHOLD` and `DISAPPROVE_THRESHOLD` cannot be 0 and greater then 5 , cause there are only five price_feeds maximum . If 
```solidity 
function setUSDThresholds(
        uint8 _approve,
        uint8 _disapprove
    ) external onlyAdmin {//@todo 
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
        APPROVE_THRESHOLD = _approve;//@audit-issue low  this cannot be more than five theretically .
        DISAPPROVE_THRESHOLD = _disapprove; // and zero  too ! 

        emit USDThresholdUpdated(_approve, _disapprove);
    }
```
link :https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L129
### Recommendation : 
```diff
 if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
+ if ( APPROVE_THRESHOLD > 5 || APPROVE_THRESHOLD == 0 ||DISAPPROVE_THRESHOLD > 5 ||  DISAPPROVE_THRESHOLD == 0  ) revert ;
        APPROVE_THRESHOLD = _approve;
        DISAPPROVE_THRESHOLD = _disapprove; 

```

