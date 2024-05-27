[G-01] The function checks are useless because a user cant set duration to be smaller than zero, it will underflow and revert.

```solidity
 @>             if (
                    uint32(block.timestamp) + uint32(_duration) <
                    lockedTokens[msg.sender][tokenContract].unlockTime
                ) {
                    revert LockDurationReducedError();
```
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L255C1-L261C18

[G-02] There is no need to put a nonReentrant on a locking mechanism

```solidity

function lockOnBehalf(
	...
@>      nonReentrant 
    {
    ...
    }
```
There is another instance here
```solidity
function lock(
	...
@>      nonReentrant 
    {
	...
    }
```

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L275-L309
