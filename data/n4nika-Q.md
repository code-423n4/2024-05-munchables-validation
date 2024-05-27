[L-01] Timestamps are capped to MAX(uint32) which will break the protocol in 82 years

All the timestamps in `LockManager.sol` like `unlockTime` and `_duration` are cast to a `uint32`. Since the times used in the protocol are in seconds and the maximum value of `uint32` is `4294967295`, this number is reached in 82 years.
When this number is reached, the protocol will break as the internally used timestamps will not work anymore.

This can easily mitigated by not downcasting timestamps to `uint32` but using `uint256` (the datatype of `block.timestamp`)