# L1 - Using only one role for priceFeed addresses instead of 5

in [LockManager.sol#L147-L154](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L147-L154) and [LockManager.sol#L181-L188](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L181-L188) there are checks to verify that `msg.sender` has one of 5 roles: `Role.PriceFeed_1`, `Role.PriceFeed_2`, `Role.PriceFeed_3`, `Role.PriceFeed_4`, `Role.PriceFeed_5`. This check is implemented in [BaseConfigStorage.sol#L38-L46](https://github.com/code-423n4/2024-05-munchables/blob/main/src/config/BaseConfigStorage.sol#L38-L46):

```
BaseConfigStorage.sol#L38-L46

modifier onlyOneOfRoles(Role[5] memory roles) {
 for (uint256 i = 0; i < roles.length; i++) {
 if (msg.sender == configStorage.getRole(roles[i])) {
 _;
 return;
 }
 }
 revert InvalidRoleError();
 }
```

As I asked to developers, the expected addresses with `Role.PriceFeed_i` are 5. So, we think is useless to define 5 different roles that are used always in the same contest. It would be better if all addresses would have a unique price feed role `Role.PriceFeed`.

# L2 - Implementing setActiveToken() and setInactiveToken() methods

The [ConfiguredToken struct](https://github.com/code-423n4/2024-05-munchables/blob/main/src/interfaces/ILockManager.sol#L17-L27) has 4 params. These params can be configured for a specific `_tokenContract` using the [configureToken() method](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L114-L127). In particular, it is not possible to configure a token with `ConfiguredToken.nftCost == 0`.

The param `ConfiguredToken.active` is a boolean that can be used to make a specific token inactive. A [modifier](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L53-L58) has been built to check it:

```
LockManager.sol#L53-L58

/// @notice Token supplied must be configured and active
modifier onlyActiveToken(address _tokenContract) {
 if (!configuredTokens[_tokenContract].active)
 revert TokenNotConfiguredError();
 _;
}
```

However, there are no methods to change just the `ConfiguredToken.active` param. This means that the admin had to use the `configureToken()` method and pass an entire `ConfiguredToken` to change just the `ConfiguredToken.active` param. This behavior can be error-prone. We suggest to implement setActiveToken() and setInactiveToken() methods:

```diff
+       function setActiveToken(address _tokenContract) external onlyAdmin onlyConfiguredToken(_tokenContract){
+           configuredTokens[_tokenContract_].active = true;
+       }
```

```diff
+       function setInctiveToken(address _tokenContract) external onlyAdmin onlyConfiguredToken(_tokenContract){
+           configuredTokens[_tokenContract_].active = true;
+       }
```

# L3 - Remove useless modifier

The [lockOnBehalf()](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L274-L291) and [lock()](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L296-L309) have two modifiers:

```
onlyConfiguredToken(_tokenContract)
onlyActiveToken(_tokenContract)
```

The first is defined in [LockManager.sol#L47-L51](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L47-L51):

```
LockManager.sol#L46-L51

/// @notice Token supplied must be configured but can be inactive
modifier onlyConfiguredToken(address _tokenContract) {
 if (configuredTokens[_tokenContract].nftCost == 0)
 revert TokenNotConfiguredError();
 _;
}
```

and the second one is defined in [LockManager.sol#L53-L58](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L53-L58)

```
LockManager.sol#L53-L58

/// @notice Token supplied must be configured and active
modifier onlyActiveToken(address _tokenContract) {
 if (!configuredTokens[_tokenContract].active)
 revert TokenNotConfiguredError();
 _;
}
```

However, according to the current implementation, there is no way to have an active token that can be not configured. So, we suggest to use just the `onlyActiveToken` modifier.
This is because the only method which can be used to update the `ConfiguredToken.active` param is the [configureToken() method](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L114-L127):

```
LockManager.sol#L114-L127

114     /// @inheritdoc ILockManager
115     function configureToken(
116         address _tokenContract,
117         ConfiguredToken memory _tokenData
118     ) external onlyAdmin {
119         if (_tokenData.nftCost == 0) revert NFTCostInvalidError();
120         if (configuredTokens[_tokenContract].nftCost == 0) {
121             // new token
122             configuredTokenContracts.push(_tokenContract);
123         }
124         configuredTokens[_tokenContract] = _tokenData;
125
126
127         emit TokenConfigured(_tokenContract, _tokenData);
128     }
```

and thanks to line 119, is not possible to have a `_tokenContract` with `ConfiguredTokens.nftCost == 0`.


# L4 - Checking the APPROVE_THRESHOLD and DISAPPROVE_THRESHOLD values to avoid deadlock

`APPROVE_THRESHOLD` and `DISAPPROVE_THRESHOLD` are `uint8` values that are used to decide when a USDPrice proposal succeeds or fails.
When a `Role.PriceFeed` address approve an USDPrice proposal using [approveUSDPrice() method](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L176-L207), if the `APPROVE_THRESHOLD` is reached, the [_execUSDPriceUpdate()](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L506-L527) is triggered to update the USD price. On the other hand, when a `Role.PriceFeed` address disapproves an USDPrice proposal using the [disapproveUSDPrice() method](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L209-L242), if the `DISAPPROVE_THRESHOLD` is reached, the USD price proposal is deleted and it is possible to propose a new USD price.

The `APPROVE_THRESHOLD` and `DISAPPROVE_THRESHOLD` values can be set using the [setUSDThresholds methos](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L129-L139). However, there is no check on their new values. Even if this method can be called only by the Admin, he/she can be unaware of the risk of having a deadlock.

Let's do an example: `DISAPPROVE_THRESHOLD = 4` and `APPROVE_THRESHOLD = 4`. Let's think we have 6 `Role.PriceFeed` addresses.
If three `Role.PriceFeed` approve the proposal and three disapprove the proposal, there is no way to reach a consensus. Furthermore, the Admin cannot change thresholds when a proposal is being evaluated. The only way to get out of this deadlock is to give the price feed role to new addresses.

We propose to have a rule on the `APPROVE_THRESHOLD` and `DISAPPROVE_THRESHOLD` values:

```
APPROVE_THRESHOLD = DISAPPROVE_THRESHOLD

Furthermore, Let's i be the number of addresses with Role.PriceFeed

Then, APPROVE_THRESHOLD + DISAPPROVE_THRESHOLD = i + 1
```

In this way, if all price feed addresses vote, there is no way to have a deadlock.


# L5 - Don't use payable(address).transfer

In the [unlock() method](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L400-L427) there is a line that permits transferring native token to the `msg.sender` which are calling the `unlock()` method:

```
LockManager.sol#L420

420         payable(msg.sender).transfer(_quantity);
```

However, there is no check on the success of the transfer operation. If the `msg.sender` is not an EOA and it doesn't implement a `payable receive/fallback` method, funds are lost.
We underline that this issue is different from the [automatic finding M-02](https://github.com/code-423n4/2024-05-munchables/blob/main/4naly3er-report.md#m-2-call-should-be-used-instead-of-transfer-on-an-address-payable) which describe that the transaction could fail due to the gas cost. Furthermore, it is not reported in [automatic finding M-03](https://github.com/code-423n4/2024-05-munchables/blob/main/4naly3er-report.md#m-3-return-values-of-transfertransferfrom-not-checked) and [automatic finding M-04](https://github.com/code-423n4/2024-05-munchables/blob/main/4naly3er-report.md#m-4-unsafe-use-of-transfertransferfrom-with-ierc20).


# L6 - Function does not remove old entries before adding new ones

The Admin role can add new configured tokens using the [configureToken() method](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L114-L127):

```
/// @inheritdoc ILockManager
function configureToken(
 address _tokenContract,
 ConfiguredToken memory _tokenData
) external onlyAdmin {
 if (_tokenData.nftCost == 0) revert NFTCostInvalidError();
 if (configuredTokens[_tokenContract].nftCost == 0) {
 // new token
 configuredTokenContracts.push(_tokenContract);
 }
 configuredTokens[_tokenContract] = _tokenData;


 emit TokenConfigured(_tokenContract, _tokenData);
}
```

However, there is no way to remove `tokenContracts` from the `configuredTokenContracts` list. Even if the `Admin` can make a certain `tokenContract` inactive, we suggest to implement a method to remove `configuredTokenContracts`.


# L7 - Using a price oracle instead of having a USDPrice proposer
The implemented mechanism to update the USDPrice is based on the agreement of several `feed addresses`. They can use the [proposeUSDPrice() method](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L141-L174) to propose a new USDPrice and the [approveUSDPrice()](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L176-L207) and [disapproveUSDPrice()](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L209-L242) methods to approve or disapprove the proposal, respectively. This mechanism is error-prone and could lead to an out-of-date USDPrice.


# [NC-1] The ```nonReentrant``` ```modifier``` should occur before all other modifiers
This is a best practice to protect against reentrancy in other modifiers

*There are 3 instances of this issue:*           

```
LockManager.sol#L275-L295

275      function lockOnBehalf(
276          address _tokenContract,
277          uint256 _quantity,
278          address _onBehalfOf
279      )
280          external
281          payable
282          notPaused
283          onlyActiveToken(_tokenContract)
284          onlyConfiguredToken(_tokenContract)
285          nonReentrant
286      {
```

```
LockManager.sol#L297-L310

297      function lock(
298          address _tokenContract,
299          uint256 _quantity
300      )
301          external
302          payable
303          notPaused
304          onlyActiveToken(_tokenContract)
305          onlyConfiguredToken(_tokenContract)
306          nonReentrant
307      {
```

```
LockManager.sol#L401-L428

401      function unlock(
402          address _tokenContract,
403          uint256 _quantity
404      ) external notPaused nonReentrant {
```