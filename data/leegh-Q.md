# Low
## [L-1]. The `tokenContract` should be validated to be alive before setting new lock duration. 
```solidity
245:    function setLockDuration(uint256 _duration) external notPaused {
246:        if (_duration > configStorage.getUint(StorageKey.MaxLockDuration))
247:            revert MaximumLockDurationError();
248:
249:        playerSettings[msg.sender].lockDuration = uint32(_duration);
240:        // update any existing lock
251:        uint256 configuredTokensLength = configuredTokenContracts.length;
252:        for (uint256 i; i < configuredTokensLength; i++) {
253:@>          address tokenContract = configuredTokenContracts[i];
254:            if (lockedTokens[msg.sender][tokenContract].quantity > 0) {
...                 //...
268:            }
269:        }
270:
271:        emit LockDuration(msg.sender, _duration);
272:    }
```
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L253



# Non Critical
## [NC-1] When copy a struct, we can copy it directly, which is more clear and simple than copy it field by field. Thus, L439-L451 can be rewritten as `LockedToken memory tmpLockedToken = lockedTokens[_player][configuredTokenContracts[i]]` for simplicity.
```solidity
Function: getLocked

438:        for (uint256 i; i < configuredTokensLength; i++) {
439:            LockedToken memory tmpLockedToken; 
440:            tmpLockedToken.unlockTime = lockedTokens[_player][
441:                configuredTokenContracts[i]
442:            ].unlockTime;
443:            tmpLockedToken.quantity = lockedTokens[_player][
444:                configuredTokenContracts[i]
445:            ].quantity;
446:            tmpLockedToken.lastLockTime = lockedTokens[_player][
447:                configuredTokenContracts[i]
448:            ].lastLockTime;
449:            tmpLockedToken.remainder = lockedTokens[_player][
450:                configuredTokenContracts[i]
451:            ].remainder;
452:            tmpLockedTokens[i] = LockedTokenWithMetadata(
453:                tmpLockedToken,
454:                configuredTokenContracts[i]
455:            );
456:        }
```
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L439-L451

## [NC-2] Redundant condition checking. When an oracle proposes a USD price, it approves the proposal by default (L167, L170). Thus `usdUpdateProposal.proposer == msg.sender` always holds when `usdUpdateProposal.approvals[msg.sender] == _usdProposalId`. So the condition in L192 is included in L194, and the former can be removed.
```solidity
Function: proposeUSDPrice
167:@>      usdUpdateProposal.proposer = msg.sender;
168:        usdUpdateProposal.proposedPrice = _price;
169:        usdUpdateProposal.contracts = _contracts;
170:@>      usdUpdateProposal.approvals[msg.sender] = _usdProposalId;

Function: approveUSDPrice

192:@>      if (usdUpdateProposal.proposer == msg.sender)
193:            revert ProposerCannotApproveError();
194:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)
195:            revert ProposalAlreadyApprovedError();
```
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L167-L170
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L192-L195

## [NC-3] Use constants for ETH address instead of address(0) to distinguish with empty address and for better readability.
```solidity
324:        if (_tokenContract == address(0)) {

374:        if (_tokenContract != address(0)) {

419:        if (_tokenContract == address(0)) {
```
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L324, 
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L374, 
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L419