### [Low-01] `_execUSDPriceUpdate()` assuming that all `tokenContacts` price which  it is going to set is same at same `price`, which is not a practically possible.

### [Low-02] In case of instant price fluctuation, A player can front-run priceFeeders and get benefited.

### [Low-03] Role.PriceFeed can `approve` & `disapprove` same time as there is no check for `disapproval` in `approveUSDPrice()`

### [Low-04] `getLocked()` should checked that a Player have locked amount for a corresponding `configuredToken`, otherwise returns data will full of empty data structures

### [Low-05] If no. of configuredTokens increase significantly then some functions like `getLocked()` will result DoS, due to exceeding block gas limit.


### [Low-06] There should be a minimum `lockDuration` check while confiuring / reconfiguring `lockDrop`

### [Low-07] No need to call `delete usdUpdateProposal` inside `proposeUSDPrice()` cause `usdUpdateProposal` already empty.

### [Low-08] Centralization
