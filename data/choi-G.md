# Gas Optimization Report: Batched Token Transfer in `LockManager` Contract

## Problem

The `LockManager` contract currently performs individual token transfers for each lock and unlock operation. This increases gas costs. By batching token transfer operations, the contract can reduce gas usage and improve overall efficiency.

### Current Implementation:

The `_lock` function in the `LockManager` contract transfers tokens individually for each lock operation. Also, the `unlock` function handles individual token transfers for each unlock operation.

```solidity
function _lock(
    address _tokenContract,
    uint256 _quantity,
    address _tokenOwner,
    address _lockRecipient
) private {

    // Transfer ERC20 tokens
    if (_tokenContract != address(0)) {
        IERC20 token = IERC20(_tokenContract);
        token.transferFrom(_tokenOwner, address(this), _quantity);
    }
}

function unlock(
    address _tokenContract,
    uint256 _quantity
) external notPaused nonReentrant {
    LockedToken storage lockedToken = lockedTokens[msg.sender][_tokenContract];
    if (lockedToken.quantity < _quantity) revert InsufficientLockAmountError();
    if (lockedToken.unlockTime > uint32(block.timestamp)) revert TokenStillLockedError();

    // Force harvest to ensure schnibbles are distributed
    accountManager.forceHarvest(msg.sender);

    lockedToken.quantity -= _quantity;

    // Transfer ERC20 tokens
    if (_tokenContract == address(0)) {
        payable(msg.sender).transfer(_quantity);
    } else {
        IERC20 token = IERC20(_tokenContract);
        token.transfer(msg.sender, _quantity);
    }

    emit Unlocked(msg.sender, _tokenContract, _quantity);
}
```

## Mitigation

- The proposed batched token transfer optimization makes a valuable upgrade to the gas efficiency of the `LockManager` contract. Implementing this optimization will reduce the gas costs and improved overall performance.

```solidity
function _lock(
    address _tokenContract,
    uint256 _quantity,
    address _tokenOwner,
    address _lockRecipient
) private {

    // Accumulate tokens for batched transfer
    if (_tokenContract != address(0)) {
        _accumulateTokens(_tokenOwner, address(this), _quantity, _tokenContract);
    }
}

function unlock(
    address _tokenContract,
    uint256 _quantity
) external notPaused nonReentrant {
    LockedToken storage lockedToken = lockedTokens[msg.sender][_tokenContract];
    if (lockedToken.quantity < _quantity) revert InsufficientLockAmountError();
    if (lockedToken.unlockTime > uint32(block.timestamp)) revert TokenStillLockedError();

    // Force harvest to ensure schnibbles are distributed
    accountManager.forceHarvest(msg.sender);

    lockedToken.quantity -= _quantity;

    // Accumulate tokens for batched transfer
    if (_tokenContract == address(0)) {
        _accumulateTokens(address(this), msg.sender, _quantity, _tokenContract);
    } else {
        _accumulateTokens(address(this), msg.sender, _quantity, _tokenContract);
    }

    emit Unlocked(msg.sender, _tokenContract, _quantity);
}

function _accumulateTokens(address from, address to, uint256 amount, address tokenContract) private {
    // Logic to accumulate token transfers
    // This can involve adding amounts to a mapping and performing the transfer in batch at the end of the transaction
}

```