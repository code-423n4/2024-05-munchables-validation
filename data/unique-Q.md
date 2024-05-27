&nbsp;

## \[L-01\] \`>\` should change it to \`>=\` to accept all days.

[0xinsanity](https://discord.com/channels/810916927919620096/1242163386058276985/1243312516067491840) mentioned that the `MaxLockDuration` is 90 days. Therefore, if someone passes 90 as the \_duration, it will revert because the check for `>` means 90 days itself is not acceptable less than 90 is acceptable like 89,88,87, and so on.

```
  function setLockDuration(uint256 _duration) external notPaused {
        if (_duration > configStorage.getUint(StorageKey.MaxLockDuration))
            revert MaximumLockDurationError();
```

should changed to `>=`

```
  if (_duration >= configStorage.getUint(StorageKey.MaxLockDuration))
```

To accept 90 also

[Mimimum lock is for 30 days...I'm locking for 90 days!](https://munchables.gitbook.io/munchables/explorers-guide/glossary/lockdrop)

## \[L-02\] Function Calls in Loop Could Lead to Denial of Service

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions.

```
function getLocked(
        address _player
    ) external view returns (LockedTokenWithMetadata[] memory _lockedTokens) {
        uint256 configuredTokensLength = configuredTokenContracts.length;
        LockedTokenWithMetadata[]
            memory tmpLockedTokens = new LockedTokenWithMetadata[](
                configuredTokensLength
            );
        for (uint256 i; i < configuredTokensLength; i++) {
            LockedToken memory tmpLockedToken;
            tmpLockedToken.unlockTime = lockedTokens[_player][
                configuredTokenContracts[i]
            ].unlockTime;
            tmpLockedToken.quantity = lockedTokens[_player][
                configuredTokenContracts[i]
            ].quantity;
            tmpLockedToken.lastLockTime = lockedTokens[_player][
                configuredTokenContracts[i]
            ].lastLockTime;
            tmpLockedToken.remainder = lockedTokens[_player][
                configuredTokenContracts[i]
            ].remainder;
            tmpLockedTokens[i] = LockedTokenWithMetadata(           <@
                tmpLockedToken,
                configuredTokenContracts[i]
            );
        }
        _lockedTokens = tmpLockedTokens;
    }
```

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L452

&nbsp;

## \[N-03\] For modern and more readable code; update import usages

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.  
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.  
*Recommendation*  
`import {contract1 , contract2} from "filename.sol";`

```
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";
import "../interfaces/ILockManager.sol";
import "../interfaces/IConfigStorage.sol";
import "../interfaces/IAccountManager.sol";
import "../interfaces/IMigrationManager.sol";
import "./BaseBlastManager.sol";
import "../interfaces/ISnuggeryManager.sol";
import "../interfaces/INFTOverlord.sol";
```

&nbsp;

## \[N-04\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

&nbsp;

## \[N-05\] NatSpec comments should be increased in contracts

> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

in complext project such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.  
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

&nbsp;

## \[N-06\] Generate perfect code headers every time

Description:  
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```