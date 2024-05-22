Summary
Unlock function handles unlocking of tokens and ETH for users. While the function uses the nonReentrant modifier the order of operations in the function can still be improved. Currently, the function transfers tokens or ETH to the caller before updating the state, which is clearly not the best practice.
Details

```solidity
function unlock(
    address _tokenContract,
    uint256 _quantity
) external notPaused nonReentrant {
    LockedToken storage lockedToken = lockedTokens[msg.sender][_tokenContract];
    if (lockedToken.quantity < _quantity)
        revert InsufficientLockAmountError();
    if (lockedToken.unlockTime > uint32(block.timestamp))
        revert TokenStillLockedError();

    // force harvest to make sure that they get the schnibbles that they are entitled to
    accountManager.forceHarvest(msg.sender);

    // Transfer token or ETH before updating state (not recommended)
    if (_tokenContract == address(0)) {
        payable(msg.sender).transfer(_quantity);
    } else {
        IERC20 token = IERC20(_tokenContract);
        token.transfer(msg.sender, _quantity);
    }

    lockedToken.quantity -= _quantity;

    emit Unlocked(msg.sender, _tokenContract, _quantity);
}
```
Vulnerability Explanation
Even though the nonReentrant modifier is used to prevent reentrancy, it's good practice to follow the checks-effects-interactions pattern. In the current implementation, the function transfers tokens or ETH to the caller before updating the state. This order of operations could be exploited if the nonReentrant modifier is ever removed or fails.

Recommended Fix
Reorder the operations in the unlock function to update the state before making any external calls. This ensures that the contract state is always updated before any transfer is made, mitigating the risk of reentrancy attacks.

Updated Function
solidity
function unlock(
    address _tokenContract,
    uint256 _quantity
) external notPaused nonReentrant {
    LockedToken storage lockedToken = lockedTokens[msg.sender][_tokenContract];
    if (lockedToken.quantity < _quantity)
        revert InsufficientLockAmountError();
    if (lockedToken.unlockTime > uint32(block.timestamp))
        revert TokenStillLockedError();

    // force harvest to make sure that they get the schnibbles that they are entitled to
    accountManager.forceHarvest(msg.sender);

    // Update state before transferring tokens
    lockedToken.quantity -= _quantity;

    // Transfer token or ETH after updating state
    if (_tokenContract == address(0)) {
        payable(msg.sender).transfer(_quantity);
    } else {
        IERC20 token = IERC20(_tokenContract);
        token.transfer(msg.sender, _quantity);
    }

    emit Unlocked(msg.sender, _tokenContract, _quantity);
}