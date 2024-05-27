### L-1
- Visibility of `LockManager.sol::playerSettings` is not mentioned which usually consider it as a bad practice. 
- Link to the code
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L26
### L-2 
- Visibility of `LockManager.sol::usdUpdateProposal` is not mentioned which usually consider it as a bad practice
- Link to the code 
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L30
### L-3
- Zero address check is Missing inside constructor 
- Link to the code 
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L60
### L-4
- No zero checks of minLockDuration in `configureLockdrop` function 
- Link to the code 
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L98
### L-5
- Missing check of `(_lockdropData.minLockDuration < uint32(configStorage.getUint(StorageKey.MaxLockDuration))`
- Link to the code 
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L98
### L-6
- Allowing to call `approveUSDPrice` function after calling `disapproveUSDPrice` function 
- Link to the code 
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L177