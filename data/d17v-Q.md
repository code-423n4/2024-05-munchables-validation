## Impact
Missing zero address check in the [LockManager::constructor](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L60-L63). This may lead to infunctional protocol if the variable addresses are updated incorrectly.

## Proof of Concept
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L60-L63

## Tools Used
Manual Review

## Recommended Mitigation Steps
Consider adding zero-address check in the constructor
```diff
constructor(address _configStorage) {
+    require(_configStorage != address(0));
    __BaseConfigStorage_setConfigStorage(_configStorage);
    _reconfigure();
}
```