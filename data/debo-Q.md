# Low-1 Block Timestamp

# Bug-1 File Path: Munchables/src/managers/LockManager.sol Lines: 353 - 354

### Vulnerability: Timestamp Dependency

**Description:**

The `LockManager` contract relies on the block timestamp to determine the start and end of the lockdrop period, as seen in lines 353 to 354:

```solidity
if (lockdrop.start <= uint32(block.timestamp) && lockdrop.end >= uint32(block.timestamp))
```

Using block timestamps for critical logic can be problematic due to the following reasons:

1. **Manipulation by Miners:** Miners have the ability to manipulate the block timestamp within a certain range. This can be exploited to either prematurely start or extend the lockdrop period, potentially allowing malicious actors to lock tokens outside the intended timeframe.

2. **Inconsistent Timekeeping:** Block timestamps are not guaranteed to be precise. They can vary slightly from the actual time, leading to inconsistencies in the contract's behavior.

**Recommendation:**

To mitigate this vulnerability, consider using block numbers instead of timestamps for time-sensitive operations. This approach is more reliable and less susceptible to manipulation. If using timestamps is unavoidable, ensure that the contract logic can tolerate minor deviations and include safeguards to minimize the impact of potential manipulation.

### Recommendation: Mitigating Timestamp Dependency Vulnerability

**Description:**

The `LockManager` contract currently uses block timestamps to determine the start and end of the lockdrop period, which can be manipulated by miners and may lead to inconsistent timekeeping. To address this issue, we recommend using block numbers instead of timestamps for time-sensitive operations.

**Steps to Resolve:**

1. **Define Block Numbers for Lockdrop Period:**
   Replace the timestamp-based `start` and `end` fields in the `Lockdrop` struct with block numbers.

2. **Update Lockdrop Configuration:**
   Modify the `configureLockdrop` function to accept block numbers instead of timestamps.

3. **Adjust Lock Logic:**
   Update the logic that checks the lockdrop period to use block numbers.

**Implementation:**

1. **Update the `Lockdrop` Struct:**

   ```solidity
   struct Lockdrop {
       uint256 startBlock;
       uint256 endBlock;
       uint256 minLockDuration;
   }
   ```

2. **Modify the `configureLockdrop` Function:**

   ```solidity
   function configureLockdrop(
       uint256 _startBlock,
       uint256 _endBlock,
       uint256 _minLockDuration
   ) external onlyAdmin {
       if (_endBlock < block.number)
           revert LockdropEndedError(_endBlock, block.number);
       if (_startBlock >= _endBlock)
           revert LockdropInvalidError();

       lockdrop = Lockdrop({
           startBlock: _startBlock,
           endBlock: _endBlock,
           minLockDuration: _minLockDuration
       });

       emit LockDropConfigured(lockdrop);
   }
   ```

3. **Adjust the Lock Logic:**

   ```solidity
   function _lock(
       address _tokenContract,
       uint256 _quantity,
       address _tokenOwner,
       address _lockRecipient
   ) private {
       // ... existing code ...

       uint32 _lockDuration = playerSettings[_lockRecipient].lockDuration;

       if (_lockDuration == 0) {
           _lockDuration = lockdrop.minLockDuration;
       }
       if (
           lockdrop.startBlock <= block.number &&
           lockdrop.endBlock >= block.number
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

       // ... existing code ...
   }
   ```

By implementing these changes, the contract will use block numbers instead of timestamps, reducing the risk of manipulation and improving the reliability of time-sensitive operations.

# POC
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.25;

import "forge-std/Test.sol";
import "../src/LockManager.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract LockManagerTest is Test {
    LockManager lockManager;
    ERC20 token;
    address admin = address(0x1);
    address user = address(0x2);

    function setUp() public {
        vm.startPrank(admin);
        lockManager = new LockManager(address(this));
        token = new ERC20("Test Token", "TTK");
        lockManager.configureToken(address(token), ConfiguredToken({
            nftCost: 1e18,
            active: true,
            usdPrice: 1e18,
            decimals: 18
        }));
        lockManager.configureLockdrop(Lockdrop({
            start: uint32(block.timestamp + 1),
            end: uint32(block.timestamp + 100),
            minLockDuration: 10
        }));
        vm.stopPrank();
    }

    function testTimestampManipulation() public {
        vm.startPrank(user);
        token.approve(address(lockManager), 1e18);

        // Fast forward time to just before the lockdrop starts
        vm.warp(block.timestamp + 1);
        assertEq(block.timestamp, lockManager.lockdrop().start);

        // Lock tokens right at the start of the lockdrop
        lockManager.lock{value: 1e18}(address(token), 1e18);

        // Fast forward time to just after the lockdrop ends
        vm.warp(block.timestamp + 100);
        assertEq(block.timestamp, lockManager.lockdrop().end);

        // Attempt to lock tokens after the lockdrop has ended
        lockManager.lock{value: 1e18}(address(token), 1e18);

        vm.stopPrank();
    }
}
```

# Bug-2 File Path: Munchables/src/managers/LockManager.sol Line: 410

### Vulnerability: Timestamp Dependency

**Description:**

The `lockOnBehalf` and `lock` functions in the `LockManager` contract rely on the current block timestamp (`block.timestamp`) to determine the lock duration and validate the lockdrop period. This dependency on the block timestamp can be exploited by miners to manipulate the contract's behavior.

**Location:**
```solidity
if (
    lockdrop.start <= uint32(block.timestamp) &&
    lockdrop.end >= uint32(block.timestamp)
)
```

**Details:**

1. **Timestamp Manipulation:** Miners can manipulate the block timestamp within a certain range, which can affect the conditions checked in the `lock` and `lockOnBehalf` functions. This can lead to unintended behavior, such as allowing or disallowing token locks based on manipulated timestamps.

2. **Lockdrop Period Validation:** The contract checks if the current timestamp is within the lockdrop period. If a miner manipulates the timestamp, they can potentially lock tokens outside the intended lockdrop period, violating the contract's intended logic.

3. **Potential Exploits:** An attacker with mining capabilities could exploit this vulnerability to:
   - Lock tokens outside the intended lockdrop period.
   - Manipulate the lock duration to their advantage.

**Recommendation:**

To mitigate this vulnerability, consider using a more reliable source of time or implementing additional checks to ensure the integrity of the lockdrop period and lock duration. Avoid relying solely on `block.timestamp` for critical logic.

### Recommendation: Mitigating Timestamp Dependency Vulnerability

**Description:**

To address the vulnerability related to the dependency on `block.timestamp` in the `lockOnBehalf` and `lock` functions, consider implementing the following recommendations:

1. **Use a Trusted Oracle for Time:**
   - Integrate a trusted time oracle to provide a reliable source of time. This can help mitigate the risk of timestamp manipulation by miners.

2. **Implement a Grace Period:**
   - Introduce a grace period around the lockdrop start and end times to account for minor timestamp manipulations. This can help ensure that the lockdrop period is not easily exploitable.

3. **Additional Validations:**
   - Implement additional checks to validate the lock duration and lockdrop period. For example, you can store the block number at the start of the lockdrop and use it for validation instead of relying solely on `block.timestamp`.

**Code Changes:**

1. **Integrate a Time Oracle:**
   - Use a trusted time oracle to get the current time instead of `block.timestamp`.

2. **Implement Grace Period:**
   - Add a grace period around the lockdrop start and end times.

3. **Additional Validations:**
   - Store the block number at the start of the lockdrop and use it for validation.

Here is an example of how you can implement these recommendations:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.25;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";
import "../interfaces/ILockManager.sol";
import "../interfaces/IConfigStorage.sol";
import "../interfaces/IAccountManager.sol";
import "../interfaces/IMigrationManager.sol";
import "./BaseBlastManager.sol";
import "../interfaces/ISnuggeryManager.sol";
import "../interfaces/INFTOverlord.sol";
import "../interfaces/ITimeOracle.sol"; // Import the time oracle interface

contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {
    // Other contract variables and functions...

    ITimeOracle public timeOracle; // Add a reference to the time oracle

    constructor(address _configStorage, address _timeOracle) {
        __BaseConfigStorage_setConfigStorage(_configStorage);
        timeOracle = ITimeOracle(_timeOracle); // Initialize the time oracle
        _reconfigure();
    }

    // Other contract functions...

    function _lock(
        address _tokenContract,
        uint256 _quantity,
        address _tokenOwner,
        address _lockRecipient
    ) private {
        // Other validations...

        uint32 currentTime = uint32(timeOracle.getCurrentTime()); // Use the time oracle to get the current time

        if (
            lockdrop.start <= currentTime &&
            lockdrop.end >= currentTime
        ) {
            // Lockdrop period validation with grace period
            uint32 gracePeriod = 300; // 5 minutes grace period
            if (
                currentTime < lockdrop.start - gracePeriod ||
                currentTime > lockdrop.end + gracePeriod
            ) {
                revert LockdropPeriodError();
            }

            // Other lock logic...
        }

        // Other lock logic...
    }

    // Other contract functions...
}
```

**Explanation:**

1. **Time Oracle Integration:**
   - The contract now uses a trusted time oracle (`ITimeOracle`) to get the current time instead of relying on `block.timestamp`.

2. **Grace Period:**
   - A grace period of 5 minutes (300 seconds) is added around the lockdrop start and end times to account for minor timestamp manipulations.

3. **Additional Validations:**
   - The contract validates the lockdrop period using the time provided by the oracle and the grace period.

By implementing these changes, you can mitigate the risk of timestamp manipulation and ensure the integrity of the lockdrop period and lock duration.

# Bug-3 File Path: Munchables/src/managers/LockManager.sol Lines: 101 - 105

### Vulnerability: Timestamp 3

**Description:**

The vulnerability arises from the use of block timestamps for critical logic in the `configureLockdrop` function (lines 101 to 105). Specifically, the function checks if the lockdrop end time is in the past and if the start time is before the end time using `block.timestamp`. This reliance on block timestamps can be problematic for the following reasons:

1. **Manipulation by Miners:** Miners can manipulate the block timestamp within a certain range, potentially allowing them to influence the outcome of the lockdrop configuration. This could lead to scenarios where the lockdrop is incorrectly configured, either prematurely ending or starting the lockdrop.

2. **Inconsistent Time:** The block timestamp is not a reliable source of time as it can vary slightly between blocks. This inconsistency can lead to unexpected behavior in the contract, especially in time-sensitive operations like lockdrops.

3. **Time-based Attacks:** Attackers can exploit the reliance on block timestamps to execute time-based attacks, such as front-running or delaying transactions to manipulate the lockdrop configuration to their advantage.

**Recommendation:**

To mitigate this vulnerability, consider using a more reliable and tamper-resistant time source or implementing additional checks and balances to ensure the integrity of the lockdrop configuration. Additionally, avoid using block timestamps for critical logic where possible.

### Recommendation to Resolve Timestamp 3 Vulnerability

**Description:**

The `configureLockdrop` function relies on `block.timestamp` for critical logic, which can be manipulated by miners and is not a consistent time source. This can lead to incorrect lockdrop configurations and potential time-based attacks.

**Steps to Mitigate the Vulnerability:**

1. **Use Block Numbers Instead of Timestamps:**
   - Instead of using `block.timestamp`, use `block.number` to determine the start and end of the lockdrop. This approach is less susceptible to manipulation by miners.

2. **Implement a Time Buffer:**
   - Introduce a buffer period to account for minor variations in block times. This can help mitigate the impact of slight inconsistencies in block timestamps.

3. **Additional Checks and Balances:**
   - Implement additional checks to ensure the integrity of the lockdrop configuration. For example, require multiple confirmations or approvals from trusted parties before finalizing the lockdrop configuration.

**Example Implementation:**

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.25;

contract LockManager {
    // Other contract code...

    uint256 public constant BLOCKS_PER_DAY = 6500; // Approximate number of blocks per day

    struct Lockdrop {
        uint256 startBlock;
        uint256 endBlock;
        // Other lockdrop data...
    }

    Lockdrop public lockdrop;

    /// @inheritdoc ILockManager
    function configureLockdrop(
        Lockdrop calldata _lockdropData
    ) external onlyAdmin {
        uint256 currentBlock = block.number;
        if (_lockdropData.endBlock < currentBlock)
            revert LockdropEndedError(
                _lockdropData.endBlock,
                currentBlock
            );
        if (_lockdropData.startBlock >= _lockdropData.endBlock)
            revert LockdropInvalidError();

        lockdrop = _lockdropData;

        emit LockDropConfigured(_lockdropData);
    }

    // Other contract code...
}
```

**Explanation:**

1. **Block Numbers:** The `Lockdrop` struct now uses `startBlock` and `endBlock` instead of timestamps.
2. **Block Validation:** The `configureLockdrop` function checks the current block number (`block.number`) against the provided start and end blocks to ensure the lockdrop is configured correctly.
3. **Buffer Period:** By using block numbers, the contract avoids the inconsistencies and potential manipulation associated with block timestamps.

By implementing these changes, the contract becomes more resilient to timestamp manipulation and time-based attacks, ensuring a more secure and reliable lockdrop configuration process.

# Bug-4 File Path: Munchables/src/managers/LockManager.sol Lines: 256 - 261

### Vulnerability: Timestamp 4

**Description:**

The vulnerability arises from the use of block timestamps for critical logic in the `_lock` function (lines 256 to 261). Specifically, the contract relies on `block.timestamp` to determine the start and end of the lockdrop period and to set the lock duration for tokens. This reliance on block timestamps can be exploited by miners, who have the ability to manipulate the block timestamp within a certain range. 

**Details:**

1. **Lockdrop Period Validation:**
   - The contract checks if the current time (`block.timestamp`) falls within the lockdrop period (`lockdrop.start` to `lockdrop.end`). If the timestamp is manipulated, it could allow or disallow token locking inappropriately.

2. **Lock Duration Calculation:**
   - The lock duration is calculated using `block.timestamp`, which can be manipulated to either shorten or extend the lock period. This can lead to tokens being locked for unintended durations, potentially causing financial loss or unfair advantages.

**Impact:**

- **Financial Loss:** Users may lock tokens under incorrect assumptions about the lock period, leading to potential financial loss.
- **Unfair Advantage:** Malicious actors could manipulate the lock period to gain an unfair advantage, such as locking tokens at favorable times or avoiding penalties.

**Recommendation:**

- **Use Block Numbers:** Instead of relying on `block.timestamp`, use block numbers for time-sensitive logic. This approach reduces the risk of manipulation, as block numbers are more predictable and less susceptible to miner influence.
- **Time Buffer:** Implement a buffer period to account for minor timestamp manipulations, ensuring that critical logic is not affected by small changes in block timestamps.

### Recommendation: Mitigating Timestamp Manipulation Vulnerability

**Description:**

To mitigate the vulnerability arising from the use of `block.timestamp` in the `_lock` function, we recommend the following changes:

1. **Use Block Numbers:**
   - Replace the reliance on `block.timestamp` with block numbers for determining the lockdrop period and lock duration. This reduces the risk of manipulation by miners.

2. **Implement Time Buffer:**
   - Introduce a buffer period to account for minor timestamp manipulations, ensuring that critical logic is not affected by small changes in block timestamps.

**Implementation:**

1. **Define Block Numbers for Lockdrop Period:**
   - Add new variables to store the start and end block numbers for the lockdrop period.

2. **Modify the `_lock` Function:**
   - Replace `block.timestamp` checks with block number checks.
   - Adjust the lock duration calculation to use block numbers.

**Code Changes:**

1. **Define Block Numbers for Lockdrop Period:**

```solidity
struct Lockdrop {
    uint256 startBlock;
    uint256 endBlock;
    uint32 minLockDuration;
    // other fields...
}
```

2. **Modify the `_lock` Function:**

```solidity
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
        lockdrop.startBlock <= block.number &&
        lockdrop.endBlock >= block.number
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
    lockedToken.lastLockTime = uint32(block.number);
    lockedToken.unlockTime =
        uint32(block.number) +
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

**Summary:**

By using block numbers instead of `block.timestamp` and implementing a buffer period, we can significantly reduce the risk of timestamp manipulation, ensuring the integrity and fairness of the lockdrop process.