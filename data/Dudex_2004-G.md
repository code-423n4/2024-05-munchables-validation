The `lock` function locks users tokens for particular `_duration`. However if user directly calls this `lock` function then his tokens `_duration` by default set as `lockdrop.minLockDuration`.
```javascript
 if (_lockDuration == 0) {
            _lockDuration = lockdrop.minLockDuration;
        }
```
 And he can not increment this `_duration` more than `MaxLockDuration` via `setLockDuration` function.
```javascript
 if (_duration > configStorage.getUint(StorageKey.MaxLockDuration))
            revert MaximumLockDurationError();
```
Which makes below check unusable in the `_lock` function
```javascript
  if (
                _lockDuration < lockdrop.minLockDuration ||
                _lockDuration >
                uint32(configStorage.getUint(StorageKey.MaxLockDuration))
            ) revert InvalidLockDurationError();
```

## Mitigation 
Please remove this unnecessary check in `_lock` function