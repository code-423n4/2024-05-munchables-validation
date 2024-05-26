# Low Issues

## L-1: Unsafe ERC20 Operations should not be used

ERC20 functions may not behave as expected. For example: return values are not always meaningful. It is recommended to use OpenZeppelin's SafeERC20 library.

<details><summary>3 Found Instances</summary>

- Found in src/managers/LockManager.sol [Line: 376](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L376)

  ```solidity
              token.transferFrom(_tokenOwner, address(this), _quantity);
  ```

- Found in src/managers/LockManager.sol [Line: 420](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L420)

  ```solidity
              payable(msg.sender).transfer(_quantity);
  ```

- Found in src/managers/LockManager.sol [Line: 423](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L423)

  ```solidity
              token.transfer(msg.sender, _quantity);
  ```

</details>

## L-2: Missing checks for `address(0)` when assigning values to address state variables

Check for `address(0)` when assigning values to address state variables.

<details><summary>2 Found Instances</summary>

- Found in src/managers/LockManager.sol [Line: 249](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L249)

  ```solidity
          playerSettings[msg.sender].lockDuration = uint32(_duration);
  ```

- Found in src/managers/LockManager.sol [Line: 265](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L265)

  ```solidity
                  lockedTokens[msg.sender][tokenContract].unlockTime =
  ```

</details>

## L-3: The `nonReentrant` `modifier` should occur before all other modifiers

This is a best-practice to protect against reentrancy in other modifiers.

<details><summary>3 Found Instances</summary>

- Found in src/managers/LockManager.sol [Line: 285](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L285)

  ```solidity
          nonReentrant
  ```

- Found in src/managers/LockManager.sol [Line: 306](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L306)

  ```solidity
          nonReentrant
  ```

- Found in src/managers/LockManager.sol [Line: 404](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L404)

  ```solidity
      ) external notPaused nonReentrant {
  ```

</details>

## L-4: Reverting ETH transfers in `fallback` and `receive` functions does not prevent forced transfers

Using `fallback` and `receive` functions to revert ETH transfers does not prevent forced transfers via `selfdestruct`. Removing these functions will reduce contract size, increase readability and maintainability, and potentially lowers gas costs.

<details><summary>2 Found Instances</summary>
- Found in src/managers/LockManager.sol [Line: 89](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L89)

```solidity
    fallback() external payable {
        revert LockManagerInvalidCallError();
    }
```

- Found in src/managers/LockManager.sol [Line: 93](https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L93)

```solidity
        receive() external payable {
        revert LockManagerInvalidCallError();
    }
```

</details>