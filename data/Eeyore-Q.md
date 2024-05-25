## [L-01] Missing `_lockdropData.minLockDuration > uint32(configStorage.getUint(StorageKey.MaxLockDuration)` check in `configureLockdrop()` function can lead to DoS during locking.

### Description

In the `configureLockdrop()` function, there is no check to prevent `lockdrop.minLockDuration` from being set higher than the system-wide `StorageKey.MaxLockDuration`. This will result in a denial of service (DoS) in the `_lock()` function when attempting to lock tokens during the lockdrop period.

### Recommendation

Add a validation check that reverts the transaction if `_lockdropData.minLockDuration > uint32(configStorage.getUint(StorageKey.MaxLockDuration))` when calling the `configureLockdrop()` function.

## [L-02] The `setLockDuration()` function can be used to increase `playerSettings.lockDuration` without relocking already unlocked tokens, leading to increased potential bonuses without lock risk.

### Description

The `playerSettings.lockDuration` is used to determine additional bonuses based on the lock duration. However, once the locked tokens have reached their unlock time, a player can increase their `lockDuration` without relocking their tokens. 

This can allow players to manipulate functionalities that depend on `playerSettings.lockDuration` to gain undeserved bonuses without the risk associated with locking their tokens.

### Recommendation

To ensure that changes to `playerSettings.lockDuration` reflect a genuine commitment to locking tokens for a longer period, any modification to `lockDuration` should require relocking the tokens. This can be achieved by implementing a mechanism that locks the tokens again for the new duration whenever `setLockDuration()` is called.

## [N-01] Redundant use of the `onlyConfiguredToken(_tokenContract)` modifier in the `lockOnBehalf()` and `lock()` functions.

### Description

The `onlyConfiguredToken(_tokenContract)` modifier checks if the specified token contract is configured. Similarly, the `onlyActiveToken(_tokenContract)` modifier, which is also applied to the same functions, implicitly ensures that the token is configured because it checks that the token is both configured and active. Therefore, using `onlyConfiguredToken(_tokenContract)` in conjunction with `onlyActiveToken(_tokenContract)` in the `lockOnBehalf()` and `lock()` functions is redundant.

### Recommendation

Remove the redundant `onlyConfiguredToken(_tokenContract)` modifier from the `lockOnBehalf()` and `lock()` functions to streamline the code and avoid unnecessary checks.