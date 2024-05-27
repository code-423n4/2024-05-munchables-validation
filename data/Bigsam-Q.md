
## QA-01: Potential Issues with the `unlock` Function

**Impact:**

This report identifies two potential issues with the `unlock` function:

1. **Unlocking Unsupported Tokens:** The code doesn't explicitly check if the token being unlocked is currently supported. This could lead to events being emitted on the blockchain for attempts to unlock tokens that are no longer supported by the contract.

2. **Unlocking Zero Quantity:** The function doesn't revert if the user tries to unlock a zero quantity of tokens. While this might not have a direct security impact, it could lead to unnecessary transactions and event emissions when combined with 1.

**Proof of Concept:**

The provided code snippet for the `unlock` function is as follows:

```solidity
/// @inheritdoc ILockManager
function unlock(
  address _tokenContract,
  uint256 _quantity
) external notPaused nonReentrant {
  LockedToken storage lockedToken = lockedTokens[msg.sender][_tokenContract];
  if (lockedToken.quantity < _quantity) revert InsufficientLockAmountError();
  if (lockedToken.unlockTime > uint32(block.timestamp)) revert TokenStillLockedError();

  // force harvest to make sure that they get the schnibbles that they are entitled to
  accountManager.forceHarvest(msg.sender);

  lockedToken.quantity -= _quantity;

  // send token
  if (_tokenContract == address(0)) {
    payable(msg.sender).transfer(_quantity);
  } else {
    IERC20 token = IERC20(_tokenContract);
    token.transfer(msg.sender, _quantity);
  }

  emit Unlocked(msg.sender, _tokenContract, _quantity);
}
```

**1. Missing Check for Active Tokens:**

- The code currently doesn't include a check like `onlyActiveToken(_tokenContract)` to ensure the token is still supported before processing the unlock request.

**2. No Reversion for Zero Quantity:**

- There's no explicit check for `_quantity > 0`.  If a user tries to unlock zero tokens, the code within the function might still execute and emit an `Unlocked` event, even though no actual unlocking occurs.

**Recommended Mitigation Steps:**

1. **Enforce Active Tokens:**
   - Apply the `onlyActiveToken` modifier to the `unlock` function. This modifier can be implemented to check if the provided `_tokenContract` address is included in a list of currently supported tokens.

2. **Revert on Zero Quantity:**
   - Modify the `unlock` function to check if `_quantity` is greater than zero. If not, revert the transaction with an appropriate error message. For example, you could add the following check at the beginning of the function:

   ```solidity
   if (_quantity == 0) revert ZeroUnlockQuantityError();
   ```

## QA-02: Redundant Check in `_execUSDPriceUpdate` Function

**Impact:**

This report identifies a potential redundancy in the `_execUSDPriceUpdate` function of the smart contract. The code performs an unnecessary check for approval threshold before executing the update logic.

**Proof of Concept:**

The code snippet reveals the following checks:

```solidity
// approveUSDPrice function (presumably)
if (usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD) {
  _execUSDPriceUpdate();
}

// _execUSDPriceUpdate function
if (usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD &&
    usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD) {
  // ... update logic
}
```

The `approveUSDPrice` function likely checks if the proposal has reached the approval threshold before calling `_execUSDPriceUpdate`. However, `_execUSDPriceUpdate` itself performs another check for the approval threshold along with a check for disapproval threshold.

**Rationale for Redundancy:**

The check in `_execUSDPriceUpdate` seems unnecessary because:

1. `_execUSDPriceUpdate` is only called only by `approveUSDPrice` after the approval threshold has already been met.
2. There's no scenario where the disapproval threshold could be reached between the `approveUSDPrice` check and the call to `_execUSDPriceUpdate`.

**Recommended Mitigation:**

- Remove the redundant check for `usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD` from the `_execUSDPriceUpdate` function. This simplifies the code and reduces gas costs associated with unnecessary checks.

## QA-03: Potential Invalid Minimum Lock Duration in `configureLockdrop`

**Impact:**

This report identifies a potential issue in the `configureLockdrop` function where the minimum lock duration for the lockdrop might be set to an invalid value.

**Proof of Concept:**

The code demonstrates how the `configureLockdrop` function sets the lockdrop details based on the provided `_lockdropData`:

```solidity
function configureLockdrop(
  Lockdrop calldata _lockdropData
) external onlyAdmin {
  // ... existing logic
  lockdrop = _lockdropData;
  // ...
}

struct Lockdrop {
  uint32 start;
  uint32 end;
  uint32 minLockDuration;
}
```

The code currently lacks a check to ensure that the `minLockDuration` is a valid value within the allowed timeframe of the lockdrop.

**Explanation:**

- `minLockDuration` is defined as a `uint32`, which can represent values from 0 to 4,294,967,295 seconds (approximately 136 years).
- The lockdrop duration itself is defined by the difference between `end` and `start`, both being `uint32` values as well.

If the `minLockDuration` is set to a value greater than the difference between `end` and `start`, it becomes impossible for users to lock their tokens for the minimum required duration within the valid lockdrop period.
**Recommended Mitigation:**

- Modify the `configureLockdrop` function to include a check that validates the `minLockDuration`:

   ```solidity
   if (_lockdropData.minLockDuration > _lockdropData.end - _lockdropData.start) {
     revert InvalidMinLockDurationError();
   }
   ```

  Or 
if it is ok, a constant minimum lock duration can be used.

