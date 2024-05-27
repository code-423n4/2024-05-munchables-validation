# Low Risk Issues
----------------------------------


## [LOW-1] Internal variables do not follow lowerCamelCase: 
Variables ```APPROVE_THRESHOLD``` and ```DISAPPROVE_THRESHOLD``` are internal variables and should follow the lowerCamelCase convention when defined.

However, this is not the case:

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L16C5-L18C36

```
uint8 APPROVE_THRESHOLD = 3;
/// @notice Threshold for removing a proposal
uint8 DISAPPROVE_THRESHOLD = 3;
```

### Recommended Mitigation Steps:

It is recommended to use lowerCamelCase for the above mentioned internal variables.


## [LOW-2] Implausible values for ```lockdrop.start``` and ```lockdrop.end``` could be set: 
The ```configureLockdrop()``` function does not check for ```lockdrop.start``` to not be zero, as well as, it does not check for ```lockdrop.end``` to be strictly greater than ```block.timestamp```:

https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L98

```
 function configureLockdrop(
        Lockdrop calldata _lockdropData
    ) external onlyAdmin {
        if (_lockdropData.end < block.timestamp)
            revert LockdropEndedError(
                _lockdropData.end,
                uint32(block.timestamp)
            ); // , "LockManager: End date is in the past");
        if (_lockdropData.start >= _lockdropData.end)
            revert LockdropInvalidError();

        lockdrop = _lockdropData;

        emit LockDropConfigured(_lockdropData);
    }
```

Zero (0) is an implausible value for ```lockdrop.start``` (since, semantically, start of a LockDrop event could not have happened on the Genesis block).

```block.timestamp``` is an implausible value for ```lockdrop.end``` (since, this would mean that the LockDrop event is only restrcited to the current block).

### Recommended Mitigation Steps:
It is recommended to modify the check in the ```configureLockdrop()``` function in the following way:
```
        if (_lockdropData.end <= block.timestamp)
            revert LockdropEndedError(
                _lockdropData.end,
                uint32(block.timestamp)
            ); // , "LockManager: End date is in the past");
        if ((_lockdropData.start >= _lockdropData.end) || (_lockdropData.start == 0))
            revert LockdropInvalidError();
 ```


## [LOW-3] The return value of ```token.transfer()``` not checked:

The ```unlock()``` function does not check the bool return value after ```token.transfer()```:

https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L423

```
    function unlock(
        address _tokenContract,
        uint256 _quantity
    ) external notPaused nonReentrant {
        LockedToken storage lockedToken = lockedTokens[msg.sender][
            _tokenContract
        ];
        if (lockedToken.quantity < _quantity)
            revert InsufficientLockAmountError();
        if (lockedToken.unlockTime > uint32(block.timestamp))
            revert TokenStillLockedError();

        // force harvest to make sure that they get the schnibbles that they are entitled to
        accountManager.forceHarvest(msg.sender);

        lockedToken.quantity -= _quantity;

        // send token
        if (_tokenContract == address(0)) {
            payable(msg.sender).transfer(_quantity);
        } else {
            IERC20 token = IERC20(_tokenContract);
            token.transfer(msg.sender, _quantity);
        }

        emit Unlocked(msg.sender, _tokenContract, _quantity);
    }
```

A situation might occur where the ```token.transfer()``` message call would fail but the ```transfer()``` function of the token contract won't revert; it would return a bool with ```false``` value. This [token behavior](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure) is in scope (as mentioned in the [README](https://code4rena.com/audits/2024-05-munchables) of the contest).

Suppose the ```token.transfer()``` returns ```false``` and this is not checked in the ```unlock()``` function. Now, in LOC-416, the ```lockedToken.quantity``` is decreased for the user:

https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L416

```
lockedToken.quantity -= _quantity;
```

This means that the user would permanently lose their tokens; they would not be able to call ```unlock()``` again and get back their tokens (which they could not get the 1st time they called ```unlock()``` due to failure of ```token.transfer()```).

This is severe as the user could not get their locked tokens back and the protocol won't be aware of it. Ultimately, the users would lose their tokens permanently.

### Recommended Mitigation Steps:

It is recommended to check for the bool return value in the above mentioned ```token.transfer()``` case.


## [LOW-4] Unnecessary use of ```storage```:

The ```_lock()``` function makes an unnecessary use of ```storage``` keyword when using ```configuredToken```:

https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L336

```
        ConfiguredToken storage configuredToken = configuredTokens[
            _tokenContract
        ];
```

The ```configuredToken``` is only referenced and not written to in the ```_lock()``` function after its definition in LOC-336.

### Recommended Mitigation Steps:

It is recommended to use ```memory``` instead of ```storage``` for ```configuredToken```.