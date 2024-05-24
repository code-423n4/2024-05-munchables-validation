## [L-1] Unnecessary ERC20 allowance check before `transferFrom(...)` function is called.

1 instance

- https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L328C13-L330C76

The `_lock(...)` function of LockManager.sol does a check to ensure the ERC20 allowance is not less than the `_quantity` parameter to be pulled with `transferFrom(...)` The allowance check is unnecessary because the `transferFrom(...)` function will revert if the allowance is not enough.

```solidity
File: LockManager.sol
function _lock(
        address _tokenContract,
        uint256 _quantity,
        address _tokenOwner,
        address _lockRecipient
    ) private {
        (
            address _mainAccount,
            MunchablesCommonLib.Player memory _player
        ) = accountManager.getPlayer(_lockRecipient);
        if (_mainAccount != _lockRecipient) revert SubAccountCannotLockError();
        if (_player.registrationDate == 0) revert AccountNotRegisteredError();
        // check approvals and value of tx matches
        if (_tokenContract == address(0)) {
            if (msg.value != _quantity) revert ETHValueIncorrectError();
        } else {
            if (msg.value != 0) revert InvalidMessageValueError();
            IERC20 token = IERC20(_tokenContract);
@>            uint256 allowance = token.allowance(_tokenOwner, address(this));
            if (allowance < _quantity) revert InsufficientAllowanceError();
        }
...//Omitted codes

373:    // Transfer erc tokens
        if (_tokenContract != address(0)) {
            IERC20 token = IERC20(_tokenContract);
376:        token.transferFrom(_tokenOwner, address(this), _quantity);
        }

```

Recommendation: Consider removing the ERC20 allowance check since `transferFrom(...)` already does the check and would revert if there is not enough allowance.


## [L-2] Users can lock and unlock zero `_quantity` of ETH 

The `lock` and `unlock` functions of the LockManager.sol do not validate the `_quantity` parameter for zero value.

Link: https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L311C5-L398C6


Recommendation: Consider validation to revert when the `_quantity` parameter of `lock` and `unlock` function is zero

