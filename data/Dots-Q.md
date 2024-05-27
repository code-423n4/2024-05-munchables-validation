# Low issues
## [L-1] Missing validation in `configureLockdrop` function
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L98C5-L112C6
Missing validation check for _lockdropData.minLockDuration.
```Solidity
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

## [L-2] Use of `setUSDThresholds` function could lead to problems in the current implementation of the code.
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L129C5-L139C6
APPROVE_THRESHOLD and DISAPPROVE_THRESHOLD shouldn't be changed if there are only 5 PriceFeed roles (The number of PriceFeed roles should be APPROVE_THRESHOLD + DISAPPROVE_THRESHOLD - 1 (assuming APPROVE_THRESHOLD = DISAPPROVE_THRESHOLD)). This could lead to problems with voting.
```Solidity
function setUSDThresholds(
        uint8 _approve,
        uint8 _disapprove
    ) external onlyAdmin {
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
        APPROVE_THRESHOLD = _approve;
        DISAPPROVE_THRESHOLD = _disapprove;

        emit USDThresholdUpdated(_approve, _disapprove);
    }
```

## [L-3] The remainder is not withdrawable
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L311-L398
When locking, if the user sends more tokens than needed for getting nfts, his excess `quantity` of tokens will be set as `remainder`. How ever in the current version of the code there is no way to withdraw this `remainder`. 

Example: 
1. The cost of 1 nft is 2e18
2. Bob sends 3e18 -> 2e18 is used to get him 1 nft and the other 1e18 is set as `remainder`.
3. Now Bob has no way of withdrawing this `remainder`. And to make use of his `remainder` he has to get 1 more nft, this time paying only 1e18.

```Solidity
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
            uint256 allowance = token.allowance(_tokenOwner, address(this));
            if (allowance < _quantity) revert InsufficientAllowanceError();
        }

        LockedToken storage lockedToken = lockedTokens[_lockRecipient][
            _tokenContract
        ];
        ConfiguredToken storage configuredToken = configuredTokens[
            _tokenContract
        ];

        // they will receive schnibbles at the new rate since last harvest if not for force harvest
        accountManager.forceHarvest(_lockRecipient);

        // add remainder from any previous lock
        uint256 quantity = _quantity + lockedToken.remainder;
        uint256 remainder;
        uint256 numberNFTs;
        uint32 _lockDuration = playerSettings[_lockRecipient].lockDuration;

        if (_lockDuration == 0) {
            _lockDuration = lockdrop.minLockDuration;
        }
        if (
            lockdrop.start <= uint32(block.timestamp) &&
            lockdrop.end >= uint32(block.timestamp)
        ) {
            if (
                _lockDuration < lockdrop.minLockDuration ||
                _lockDuration >
                uint32(configStorage.getUint(StorageKey.MaxLockDuration))
            ) revert InvalidLockDurationError();
            if (msg.sender != address(migrationManager)) {
                // calculate number of nfts
                remainder = quantity % configuredToken.nftCost;
                numberNFTs = (quantity - remainder) / configuredToken.nftCost;

                if (numberNFTs > type(uint16).max) revert TooManyNFTsError();

                // Tell nftOverlord that the player has new unopened Munchables
                nftOverlord.addReveal(_lockRecipient, uint16(numberNFTs));
            }
        }

        // Transfer erc tokens
        if (_tokenContract != address(0)) {
            IERC20 token = IERC20(_tokenContract);
            token.transferFrom(_tokenOwner, address(this), _quantity);
        }

        lockedToken.remainder = remainder;
        lockedToken.quantity += _quantity;
        lockedToken.lastLockTime = uint32(block.timestamp);
        lockedToken.unlockTime =
            uint32(block.timestamp) +
            uint32(_lockDuration);

        // set their lock duration in playerSettings
        playerSettings[_lockRecipient].lockDuration = _lockDuration;

        emit Locked(
            _lockRecipient,
            _tokenOwner,
            _tokenContract,
            _quantity,
            remainder,
            numberNFTs,
            _lockDuration
        );
    }
```