## [L-01] Users who go over the max lock duration by mistake cannot reset/reduce their lock durations in order to lock tokens as it would revert

When a user goes over the max lock duration by mistake, they are forced to make a new account with another wallet in order to lock tokens. 

TLDR: user can breach the max lock duration by calling the setter function twice or more times with arguments close to the max seconds value. Since, they cannot reduce the lock duration, they are forced to make a new account in order to lock

[LockManager.sol#L245-L272](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L245-L272)
[LockManager.sol#L356-L360](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L356-L360)

```solidity
function _lock(
        address _tokenContract,
        uint256 _quantity,
        address _tokenOwner,
        address _lockRecipient
    ) private {
        ...
        uint32 _lockDuration = playerSettings[_lockRecipient].lockDuration;

        if (_lockDuration == 0) {
            _lockDuration = lockdrop.minLockDuration;
        }
        if (
            lockdrop.start <= uint32(block.timestamp) &&
            lockdrop.end >= uint32(block.timestamp)
        ) {
            if (
 @>               _lockDuration < lockdrop.minLockDuration ||
                _lockDuration >
                uint32(configStorage.getUint(StorageKey.MaxLockDuration))
            ) revert InvalidLockDurationError();
            ...
        }

        ...
    }
```

Enforce users cannot breach the max duration in the first place by checking the new duration plus the previously set one doesn't exceed the max before mutating the users' `playerSettings` duration

## [L-02] Locking will not be possible past 2106 as uint32 would overflow

This would affect the implementations of setting up a lock duration, locking and unlocking tokens.

[LockManager.sol#L249](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L249)
[LockManager.sol#L265](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L265)
[LockManager.sol#L353-L354](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L353-L354)
[LockManager.sol#L381-L384](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L381-L384)
[LockManager.sol#L410](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L410)

Sample snippet from one of the many lines of instances:

```solidity
SNIPPET FROM _lock() implementation:

lockedToken.lastLockTime = uint32(block.timestamp);
        lockedToken.unlockTime =
            uint32(block.timestamp) +
            uint32(_lockDuration);
```

Using a bigger data size type would be sufficient e.g uint64 or 48.

## [L-03] USDB, ETH, WETH etc can both have the same price in a certain case

Calling `proposeUSDPrice()` will set all the supported token's price to the same thing even though these tokens in reality don't have the same price e.g USDB is $1 and WETH is $3700

[LockManager.sol#L142-L174](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L142-L174)
[LockManager.sol#L20](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L203)

```solidity
function _execUSDPriceUpdate() internal {
        if (
            usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD &&
            usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD
        ) {
            uint256 updateTokensLength = usdUpdateProposal.contracts.length;
            for (uint256 i; i < updateTokensLength; i++) {
                address tokenContract = usdUpdateProposal.contracts[i];
                if (configuredTokens[tokenContract].nftCost != 0) {
                    configuredTokens[tokenContract].usdPrice = usdUpdateProposal
                        .proposedPrice;

                    emit USDPriceUpdated(
                        tokenContract,
                        usdUpdateProposal.proposedPrice
                    );
                }
            }

            delete usdUpdateProposal;
        }
    }
```

Instead of updating all of the prices in one go, update specific token's price and use the `proposeUSDPrice()` argument `address[] calldata _contracts` to determine which supported token price to update.

## [L-04] Use safeTransfer for USDB & WETH or ERC20 token unlocks

[LockManager.sol#L421-L424](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L421-L424)

```solidity
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
   @>       token.transfer(msg.sender, _quantity);
        }

        emit Unlocked(msg.sender, _tokenContract, _quantity);
    }

```

## [Gov-01] Token prices mutation for the supported tokens can be done even when the contract is paused

When the contract is paused, token price proposals, approvals and disapprovals can still be carried out. Since the other dependent contracts on LockManager querying USD value functions for example, `getLockedWeightedValue` can still be reached, it would make sense to disable those mutations when the contract is paused

 [LockManager.sol#L177-L207](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L177-L207)
 [LockManager.sol#L210-L242](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L210-L242)