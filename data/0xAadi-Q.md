# [L-01] Enforce a Minimum `lockDuration` in `configureLockdrop`
The `configureLockdrop` function in the `LockManager` contract lacks enforcement of a minimum `lockDuration`. This could potentially allow setting a `lockDuration` that is too short, undermining the intended functionality and security of the lockdrop mechanism.

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L98C4-L112C6

By not enforcing a minimum `lockDuration`, the contract is susceptible to configurations that may not provide sufficient time for the intended lockdrop benefits to materialize.

# [L-02] Disallow Lockdrop Updates After Lockdrop Has Started
The `LockManager` contract currently allows for the lockdrop configuration to be updated even after the lockdrop has started. This could lead to inconsistencies and potential manipulation of the lockdrop process.

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L98C4-L112C6

Allowing updates to the lockdrop configuration after it has started can result in several issues:
- **Manipulation Risk:** Malicious actors could exploit this to alter the lockdrop parameters in their favor after users have already committed funds.
- **User Distrust:** Users might lose trust in the system if they perceive that lockdrop parameters can change unpredictably, affecting their expected rewards.
- **Inconsistent Behavior:** Changes to lockdrop parameters mid-process can lead to unexpected and inconsistent behaviors, undermining the reliability of the lockdrop mechanism.

# [L-03] Enforce Token Decimals to Be 18 or Less in `configureToken`
The `configureToken` function in the `LockManager` contract currently does not enforce a limit on the number of decimals a token can have. This can lead to potential issues with token operations and calculations that assume a standard decimal value, typically 18.

Not enforcing a maximum decimal limit of 18 can problematic in `getLockedWeightedValue()`

```solidity
    // We are assuming all tokens have a maximum of 18 decimals and that USD Price is denoted in 1e18
    uint256 deltaDecimal = 10 **
        (18 -
            configuredTokens[configuredTokenContracts[i]].decimals);
    lockedWeighted +=
        (deltaDecimal *
            lockedTokens[_player][configuredTokenContracts[i]]
                .quantity *
            configuredTokens[configuredTokenContracts[i]]
                .usdPrice) /
        1e18;
```
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L472C17-L482C26

- **Calculation Errors:** Tokens with more than 18 decimals can cause overflows and inaccuracies in arithmetic calculations in `getLockedWeightedValue`. Which expect all tokens have a decimal which is <=18

# [L-04] Restrict Setting `APPROVE_THRESHOLD` and `DISAPPROVE_THRESHOLD` to Prevent Zero Values
In the `LockManager` contract, `APPROVE_THRESHOLD` and `DISAPPROVE_THRESHOLD` are used to determine the number of approvals and disapprovals required for certain actions. However, the current implementation does not restrict these thresholds from being set to zero, which can undermine the integrity of the approval process.

```solidity
    function setUSDThresholds(
        uint8 _approve,
        uint8 _disapprove
    ) external onlyAdmin {
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
        APPROVE_THRESHOLD = _approve;
        DISAPPROVE_THRESHOLD = _disapprove;


        emit USDThresholdUpdated(_approve, _disapprove);
    }
```
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L129C5-L139C6

Allowing `APPROVE_THRESHOLD` and `DISAPPROVE_THRESHOLD` to be set to zero can have serious implications:

- **Bypassing Approval Process:** If `APPROVE_THRESHOLD` is set to zero, proposals can be approved without any actual approvals, effectively bypassing the intended process.
- **Disabling Disapproval Mechanism:** If `DISAPPROVE_THRESHOLD` is set to zero, disapprovals will be rendered meaningless, as any proposal can be disapproved without any valid disapprovals.