# Title : Confusing LockDuration Event When No Tokens Configured

## Risk : Low

## Impact
In `LockManager::setLockDuration` function, if `configuredTokensLength` is 0, the for loop is skipped and `LockDuration` event is emited, which could result to confusion. 

## Vulnerability Details
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L251
```
function setLockDuration(uint256 _duration) external notPaused {
        if (_duration > configStorage.getUint(StorageKey.MaxLockDuration))
            revert MaximumLockDurationError();

        playerSettings[msg.sender].lockDuration = uint32(_duration);
        // update any existing lock
@>        uint256 configuredTokensLength = configuredTokenContracts.length;
//@audit if configuredTokensLength is 0 the for loop is skipped
@>        for (uint256 i; i < configuredTokensLength; i++) {
            address tokenContract = configuredTokenContracts[i];
            if (lockedTokens[msg.sender][tokenContract].quantity > 0) {
                // check they are not setting lock time before current unlocktime
                if (
                    uint32(block.timestamp) + uint32(_duration) <
                    lockedTokens[msg.sender][tokenContract].unlockTime
                ) {
                    revert LockDurationReducedError();
                }

                uint32 lastLockTime = lockedTokens[msg.sender][tokenContract]
                    .lastLockTime;
                lockedTokens[msg.sender][tokenContract].unlockTime =
                    lastLockTime +
                    uint32(_duration);
            }
        }
//@audit The event LockDuration is emitted even if the for loop does not execute.
@>        emit LockDuration(msg.sender, _duration);
    }
```

## Recommendation
Add a check `configuredTokensLength > 0` before the for loop