## [L-01] `remainder` of tokens locked can be stuck without external help.

The `LockManager::_lock` function is as follows ( [https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L311](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L311) ):

```solidity
    function _lock(
        address _tokenContract,
        uint256 _quantity,
        address _tokenOwner,
        address _lockRecipient
    ) private {
        ...
        LockedToken storage lockedToken = lockedTokens[_lockRecipient][
            _tokenContract
        ];
        ConfiguredToken storage configuredToken = configuredTokens[
            _tokenContract
        ];
        ...
        if (
            lockdrop.start <= uint32(block.timestamp) &&
            lockdrop.end >= uint32(block.timestamp)
        ) {
            ...
            if (msg.sender != address(migrationManager)) {
                // calculate number of nfts
@-->            remainder = quantity % configuredToken.nftCost;
                numberNFTs = (quantity - remainder) / configuredToken.nftCost;

                if (numberNFTs > type(uint16).max) revert TooManyNFTsError();

                // Tell nftOverlord that the player has new unopened Munchables
                nftOverlord.addReveal(_lockRecipient, uint16(numberNFTs));
            }
        }
        ...
@-->    lockedToken.remainder = remainder;
        lockedToken.quantity += _quantity;
        ...
    }
```

Here, if the `quantity` of tokens locked is not a multiple of `configuredToken.nftCost`, then `remainder` will be greater than 0. And this `remainder` will be stored in `lockedToken.remainder` variable. But, this `remainder` will be stuck in the contract unless more tokens are locked by the user to make the `quantity` a multiple of `configuredToken.nftCost`. `remainder` can't be withdrawn even by unlocking the tokens. Thus, if a user doesn't have more tokens to lock to make the `quantity` a multiple of `configuredToken.nftCost`, then the `remainder` will be stuck in the contract unless external help is provided to user by providing them more tokens to lock.

For reference, `LockManager::unlock` function is as follows ( [https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L401](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L401) ):

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

@-->    lockedToken.quantity -= _quantity;

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

Here, it can be seen that `lockedToken.remainder` is not considered while unlocking the tokens.

Ideally, there should be a function to withdraw the `remainder` tokens from the contract.
