1)Unchecked External Calls

IERC20 token = IERC20(_tokenContract);
token.transferFrom(_tokenOwner, address(this), _quantity); // in _lock function
token.transfer(msg.sender, _quantity); // in unlock function

2)configureLockdrop Function 
Risk: The function does not validate whether the lockdrop data passed is valid within the context of the contract’s existing state.
Recommendation: Add checks to ensure lockdrop can only be configured during specific contract states or within certain conditions.


function configureLockdrop(Lockdrop calldata _lockdropData) external onlyAdmin {
    if (_lockdropData.end < block.timestamp)
        revert LockdropEndedError(_lockdropData.end, uint32(block.timestamp));
    if (_lockdropData.start >= _lockdropData.end)
        revert LockdropInvalidError();

    lockdrop = _lockdropData;

    emit LockDropConfigured(_lockdropData);
}

3)configureToken Function
Risk: No check is performed to ensure the token contract address is valid and not a zero address.
Recommendation: Add a check to ensure _tokenContract is not zero.

4)proposeUSDPrice, approveUSDPrice, disapproveUSDPrice Functions
Risk: These functions could be subject to manipulation if the roles and thresholds are not properly managed or if the proposal process has logical flaws.
Recommendation: Ensure that roles are managed securely and consider edge cases where multiple proposals might interfere with each other.
Role Manipulation and Mismanagement:

The current implementation uses onlyOneOfRoles modifier to restrict access to certain functions. However, the modifier and the role management system might not be secure enough to prevent unauthorized access.
Example issue: If a user somehow obtains a role or if role management has flaws, they can propose or approve USD price updates.
Proposal Process Logical Flaws:

The current proposal process allows only one active proposal at a time, but there is no unique ID to ensure the integrity of each proposal.
Example issue: If a proposal is in progress and another proposal is attempted, it might overwrite or interfere with the current proposal, causing logical flaws.
Approval and Disapproval Process:

The current implementation does not handle cases where multiple approvals or disapprovals can be manipulated by the same user.
Example issue: A user might approve or disapprove the same proposal multiple times, leading to manipulation of the approval/disapproval count.

proposeUSDPrice Function: Add unique proposal ID tracking and secure role management.

approveUSDPrice Function: Add checks to prevent multiple approvals from the same user.



5)setLockDuration Function
Risk: The function can become expensive if configuredTokenContracts is large because it loops through the entire array.
Recommendation: Limit the size of configuredTokenContracts or refactor the loop to be more efficient.

6)_lock Function
Risk: This function contains several branches and checks that can be complex to maintain and audit.
Recommendation: Refactor the function to break down complex logic into smaller, more manageable internal functions

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

    LockedToken storage lockedToken = lockedTokens[_lockRecipient][_tokenContract];
    ConfiguredToken storage configuredToken = configuredTokens[_tokenContract];

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
            _lockDuration > uint32(configStorage.getUint(StorageKey.MaxLockDuration))
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
    lockedToken.unlockTime = uint32(block.timestamp) + uint32(_lockDuration);

    // set their lock duration in playerSettings
    playerSettings[_lockRecipient].lockDuration = _lockDuration;

    emit Locked(
        _lockRecipient,
        _tokenOwner,
        _tokenContract,
        _quantity,

7)Unchecked Return Values
Risk: The transfer functions (token.transferFrom and token.transfer) do not check the return value, which might fail silently.
Recommendation: Use OpenZeppelin’s SafeERC20 library to handle ERC20 operations safely.

