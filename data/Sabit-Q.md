1. There is no check that lock duration is not zero:

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L245

In setLockDuration(), _duration parameter can be set to zero and the function won't revert. Instead, it wouold emit `LockDuration(msg.sender, _duration);`

Recommendation:
Check _duration against 0.