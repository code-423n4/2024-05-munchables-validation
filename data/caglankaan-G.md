### Prevent re-setting a state variable with the same value
Not only is wasteful in terms of gas, but this is especially problematic when an event is emitted and the old and new values set are the same, as listeners might not expect this kind of scenario.

```solidity
Path: ./src/managers/LockManager.sol

109:        lockdrop = _lockdropData;	// @audit-issue

124:        configuredTokens[_tokenContract] = _tokenData;	// @audit-issue

135:        APPROVE_THRESHOLD = _approve;	// @audit-issue

136:        DISAPPROVE_THRESHOLD = _disapprove;	// @audit-issue

168:        usdUpdateProposal.proposedPrice = _price;	// @audit-issue

169:        usdUpdateProposal.contracts = _contracts;	// @audit-issue
```
[109](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L109-L109), [124](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L124-L124), [135](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L135-L135), [136](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L136-L136), [168](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L168-L168), [169](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L169-L169), 


#### Recommendation

Implement checks in your Solidity contracts to avoid re-setting state variables to their existing values. Prior to updating a state variable, compare the new value with the current value and proceed with the assignment only if they differ. Additionally, ensure that events related to state variable updates are emitted only when actual changes occur. This approach not only saves gas but also prevents confusion and unnecessary triggers in event listeners. Regularly reviewing and optimizing your contract for such redundancies can significantly enhance efficiency and clarity in contract operations.

### State Variable Access Within a Loop
State variable reads and writes are more expensive than local variable reads and writes. Therefore, it is recommended to replace state variable reads and writes within loops with a local variable. Gas savings should be multiplied by the average loop length.

```solidity
Path: ./src/managers/LockManager.sol

253:            address tokenContract = configuredTokenContracts[i];	// @audit-issue: `configuredTokenContracts` used in loop.

254:            if (lockedTokens[msg.sender][tokenContract].quantity > 0) {	// @audit-issue: `lockedTokens` used in loop.

258:                    lockedTokens[msg.sender][tokenContract].unlockTime	// @audit-issue: `lockedTokens` used in loop.

263:                uint32 lastLockTime = lockedTokens[msg.sender][tokenContract]	// @audit-issue: `lockedTokens` used in loop.

265:                lockedTokens[msg.sender][tokenContract].unlockTime =	// @audit-issue: `lockedTokens` used in loop.

440:            tmpLockedToken.unlockTime = lockedTokens[_player][	// @audit-issue: `lockedTokens` used in loop.

441:                configuredTokenContracts[i]	// @audit-issue: `configuredTokenContracts` used in loop.

443:            tmpLockedToken.quantity = lockedTokens[_player][	// @audit-issue: `lockedTokens` used in loop.

444:                configuredTokenContracts[i]	// @audit-issue: `configuredTokenContracts` used in loop.

446:            tmpLockedToken.lastLockTime = lockedTokens[_player][	// @audit-issue: `lockedTokens` used in loop.

447:                configuredTokenContracts[i]	// @audit-issue: `configuredTokenContracts` used in loop.

449:            tmpLockedToken.remainder = lockedTokens[_player][	// @audit-issue: `lockedTokens` used in loop.

450:                configuredTokenContracts[i]	// @audit-issue: `configuredTokenContracts` used in loop.

454:                configuredTokenContracts[i]	// @audit-issue: `configuredTokenContracts` used in loop.

468:                lockedTokens[_player][configuredTokenContracts[i]].quantity >	// @audit-issue: `lockedTokens` used in loop.

468:                lockedTokens[_player][configuredTokenContracts[i]].quantity >	// @audit-issue: `configuredTokenContracts` used in loop.

470:                configuredTokens[configuredTokenContracts[i]].active	// @audit-issue: `configuredTokens` used in loop.

470:                configuredTokens[configuredTokenContracts[i]].active	// @audit-issue: `configuredTokenContracts` used in loop.

475:                        configuredTokens[configuredTokenContracts[i]].decimals);	// @audit-issue: `configuredTokens` used in loop.

475:                        configuredTokens[configuredTokenContracts[i]].decimals);	// @audit-issue: `configuredTokenContracts` used in loop.

478:                        lockedTokens[_player][configuredTokenContracts[i]]	// @audit-issue: `lockedTokens` used in loop.

478:                        lockedTokens[_player][configuredTokenContracts[i]]	// @audit-issue: `configuredTokenContracts` used in loop.

480:                        configuredTokens[configuredTokenContracts[i]]	// @audit-issue: `configuredTokens` used in loop.

480:                        configuredTokens[configuredTokenContracts[i]]	// @audit-issue: `configuredTokenContracts` used in loop.

513:                address tokenContract = usdUpdateProposal.contracts[i];	// @audit-issue: `usdUpdateProposal` used in loop.

514:                if (configuredTokens[tokenContract].nftCost != 0) {	// @audit-issue: `configuredTokens` used in loop.

515:                    configuredTokens[tokenContract].usdPrice = usdUpdateProposal	// @audit-issue: `configuredTokens` used in loop.

515:                    configuredTokens[tokenContract].usdPrice = usdUpdateProposal	// @audit-issue: `usdUpdateProposal` used in loop.

520:                        usdUpdateProposal.proposedPrice	// @audit-issue: `usdUpdateProposal` used in loop.
```
[253](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L253-L253), [254](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L254-L254), [258](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L258-L258), [263](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L263-L263), [265](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L265-L265), [440](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L440-L440), [441](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L441-L441), [443](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L443-L443), [444](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L444-L444), [446](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L446-L446), [447](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L447-L447), [449](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L449-L449), [450](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L450-L450), [454](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L454-L454), [468](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L468-L468), [468](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L468-L468), [470](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L470-L470), [470](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L470-L470), [475](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L475-L475), [475](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L475-L475), [478](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L478-L478), [478](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L478-L478), [480](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L480-L480), [480](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L480-L480), [513](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L513-L513), [514](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L514-L514), [515](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L515-L515), [515](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L515-L515), [520](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L520-L520), 


#### Recommendation

Optimize gas usage in Solidity by replacing state variable reads and writes within loops with local variable operations. This reduces gas costs significantly, especially when multiplied by the average loop length.

### Unnecessary casting as variable is already of the same type
In Solidity, explicitly casting a variable to a type that it already represents is redundant and can lead to confusion and clutter in the code. This unnecessary casting doesn't typically consume additional gas since Solidity's optimizer often removes such redundant conversions during compilation. However, it does affect code readability and may obscure the actual intent of the code, making it harder for developers to understand and maintain. Ensuring that casting is used only when necessary helps maintain clean, clear, and efficient code.

```solidity
Path: ./src/managers/LockManager.sol

384:            uint32(_lockDuration);	// @audit-issue: Variable `_lockDuration` is converted to `uint32` from type `uint32`.
```
[384](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L384-L384), 


#### Recommendation

Review your Solidity code for instances of unnecessary casting where variables are cast to their own type. Remove these redundant casts to enhance code clarity and maintainability. When writing new code, ensure that casting is only applied when changing a variable's type is genuinely needed. This practice helps in keeping the codebase straightforward and understandable, reducing potential confusion and errors associated with misinterpreting the variable types.

### Same cast is done multiple times
It's cheaper to do it once, and store the result to a variable.

```solidity
Path: ./src/managers/LockManager.sol

249:        playerSettings[msg.sender].lockDuration = uint32(_duration);	// @audit-issue: Same casting also done in line(s): `[267, 257]`.

353:            lockdrop.start <= uint32(block.timestamp) &&	// @audit-issue: Same casting also done in line(s): `[354, 381, 383]`.
```
[249](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L249-L249), [353](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L353-L353), 


#### Recommendation

Identify instances in your Solidity contracts where the same value is cast to another type multiple times. Refactor these operations by performing the cast once and storing the result in a local variable. Use this variable for all subsequent operations requiring the cast value.

### `address(this)` should be cached
Cacheing saves gas when compared to repeating the calculation at each point it is used in the contract.

```solidity
Path: ./src/managers/LockManager.sol

376:            token.transferFrom(_tokenOwner, address(this), _quantity);	// @audit-issue: `adress(this)` also used on line(s): [329]
```
[376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L376-L376), 


#### Recommendation

To enhance gas efficiency, cache the contract's address by storing `address(this)` in a state variable at the point of contract deployment or initialization. Use this cached address throughout the contract instead of repeatedly calling `address(this)`. This practice reduces the gas cost associated with multiple computations of the contract's address, leading to more efficient contract execution, especially in scenarios with frequent usage of the contract's address.

### `msg.value` should be cached
Cacheing saves gas when compared to repeating the calculation at each point it is used in the contract.

```solidity
Path: ./src/managers/LockManager.sol

325:            if (msg.value != _quantity) revert ETHValueIncorrectError();	// @audit-issue: `msg.value` also used on line(s): [327]
```
[325](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L325-L325), 


#### Recommendation

Consider caching `msg.value` in a local variable at the start of payable functions where it is accessed multiple times. Implement this optimization by assigning `msg.value` to a local variable and using this variable for subsequent operations within the function. For example:
        ```solidity
        function myPayableFunction() public payable {
            uint256 cachedValue = msg.value;
            // Use cachedValue instead of msg.value for further operations
        }


### Using Prefix Operators Costs Less Gas Than Postfix Operators in Loops
Conditions can be optimized issues in Solidity refer to situations where smart contract developers write conditional statements that can be simplified or optimized for better gas efficiency, readability, and maintainability. Optimizing conditions can lead to more cost-effective and secure smart contracts.

```solidity
Path: ./src/managers/LockManager.sol

171:        usdUpdateProposal.approvalsCount++;	// @audit-issue

200:        usdUpdateProposal.approvalsCount++;	// @audit-issue

232:        usdUpdateProposal.disapprovalsCount++;	// @audit-issue

252:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

438:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

466:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

512:            for (uint256 i; i < updateTokensLength; i++) {	// @audit-issue
```
[171](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L171-L171), [200](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L200-L200), [232](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L232-L232), [252](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L252-L252), [438](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L438-L438), [466](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L466-L466), [512](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L512-L512), 


#### Recommendation

To improve gas efficiency in your Solidity code, prefer using prefix operators (e.g., `++i` or `--i`) instead of postfix operators (e.g., `i++` or `i--`) within loops. Prefix operators typically result in lower gas costs and can contribute to more efficient contract execution.

### Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

```solidity
Path: ./src/managers/LockManager.sol

263:                uint32 lastLockTime = lockedTokens[msg.sender][tokenContract]	// @audit-issue: Same index access of variable `lockedTokens` is used also on line(s): ['258', '254'].

443:            tmpLockedToken.quantity = lockedTokens[_player][	// @audit-issue: Same index access of variable `lockedTokens` is used also on line(s): ['449', '440', '446'].

447:                configuredTokenContracts[i]	// @audit-issue: Same index access of variable `configuredTokenContracts` is used also on line(s): ['450', '454', '441', '444'].

478:                        lockedTokens[_player][configuredTokenContracts[i]]	// @audit-issue: Same index access of variable `lockedTokens` is used also on line(s): ['468'].

478:                        lockedTokens[_player][configuredTokenContracts[i]]	// @audit-issue: Same index access of variable `configuredTokenContracts` is used also on line(s): ['480', '475', '468', '470'].

475:                        configuredTokens[configuredTokenContracts[i]].decimals);	// @audit-issue: Same index access of variable `configuredTokens` is used also on line(s): ['470', '480'].
```
[263](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L263-L263), [443](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L443-L443), [447](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L447-L447), [478](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L478-L478), [478](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L478-L478), [475](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L475-L475), 


#### Recommendation

When a mapping or array value is accessed multiple times within a function, cache it in a local `storage` or `calldata` variable. This approach minimizes the gas cost by reducing the number of hash computations for mappings and offset calculations for arrays. Carefully review your functions to identify opportunities for such optimizations, especially in functions with frequent or repeated accesses to the same mapping or array element, to enhance gas efficiency in your contracts.

### State Variables That Are Used Multiple Times In a Function Should Be Cached In Stack Variables
When performing multiple operations on a state variable in a function, it is recommended to cache it first. Either multiple reads or multiple writes to a state variable can save gas by caching it on the stack. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. Saves 100 gas per instance.


```solidity
Path: ./src/managers/LockManager.sol

199:        usdUpdateProposal.approvals[msg.sender] = _usdProposalId;	// @audit-issue: State variable `_usdProposalId` is used also on line(s): ['194'].

191:        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();	// @audit-issue: State variable `usdUpdateProposal` is used also on line(s): ['192'].

233:        usdUpdateProposal.disapprovals[msg.sender] = _usdProposalId;	// @audit-issue: State variable `_usdProposalId` is used also on line(s): ['227', '225'].

263:                uint32 lastLockTime = lockedTokens[msg.sender][tokenContract]	// @audit-issue: State variable `lockedTokens` is used also on line(s): ['258', '254'].

357:                _lockDuration < lockdrop.minLockDuration ||	// @audit-issue: State variable `lockdrop` is used also on line(s): ['350'].

364:                numberNFTs = (quantity - remainder) / configuredToken.nftCost;	// @audit-issue: State variable `configuredToken` is used also on line(s): ['363'].

443:            tmpLockedToken.quantity = lockedTokens[_player][	// @audit-issue: State variable `lockedTokens` is used also on line(s): ['449', '440', '446'].

447:                configuredTokenContracts[i]	// @audit-issue: State variable `configuredTokenContracts` is used also on line(s): ['450', '454', '441', '444'].

478:                        lockedTokens[_player][configuredTokenContracts[i]]	// @audit-issue: State variable `lockedTokens` is used also on line(s): ['468'].

478:                        lockedTokens[_player][configuredTokenContracts[i]]	// @audit-issue: State variable `configuredTokenContracts` is used also on line(s): ['480', '475', '468', '470'].

475:                        configuredTokens[configuredTokenContracts[i]].decimals);	// @audit-issue: State variable `configuredTokens` is used also on line(s): ['470', '480'].

513:                address tokenContract = usdUpdateProposal.contracts[i];	// @audit-issue: State variable `usdUpdateProposal` is used also on line(s): ['511'].

520:                        usdUpdateProposal.proposedPrice	// @audit-issue: State variable `usdUpdateProposal` is used also on line(s): ['515'].
```
[199](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L199-L199), [191](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L191-L191), [233](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L233-L233), [263](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L263-L263), [357](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L357-L357), [364](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L364-L364), [443](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L443-L443), [447](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L447-L447), [478](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L478-L478), [478](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L478-L478), [475](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L475-L475), [513](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L513-L513), [520](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L520-L520), 


#### Recommendation

Cache state variables in stack or local memory variables within functions when they are used multiple times. This approach replaces costlier Gwarmaccess operations with cheaper stack reads, saving approximately 100 gas per instance and optimizing overall contract performance.

### Divisions can be unchecked to save gas
The expression type(int).min/(-1) is the only case where division causes an overflow. Therefore, uncheck can be used to save gas in scenarios where it is certain that such an overflow will not occur.

```solidity
Path: ./src/managers/LockManager.sol

364:                numberNFTs = (quantity - remainder) / configuredToken.nftCost;	// @audit-issue

477:                    (deltaDecimal *	// @audit-issue
```
[364](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L364-L364), [477](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L477-L477), 


#### Recommendation

Utilize 'unchecked' blocks in Solidity for divisions where overflow is impossible, such as when 'type(int).min/(-1)' is not a concern. This can save gas by bypassing overflow checks in these specific cases.

### Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement
Unchecked keyword can be added to such scenerios: 
`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`

```solidity
Path: ./src/managers/LockManager.sol

416:        lockedToken.quantity -= _quantity;	// @audit-issue
```
[416](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L416-L416), 


#### Recommendation

In scenarios where subtraction cannot result in underflow due to prior `require()` or `if`-statements, wrap these operations in an `unchecked` block to save gas. This optimization should only be applied when the safety of the operation is assured. Carefully analyze each case to confirm that underflow is impossible before implementing `unchecked` blocks, as incorrect usage can lead to vulnerabilities in the contract.

### Stack variable is only used once
If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the 3 gas the extra stack assignment would spend

```solidity
Path: ./src/managers/LockManager.sol

287:        address tokenOwner = msg.sender;	// @audit-issue: tokenOwner used only on line: 293

317:        (	// @audit-issue: _mainAccount used only on line: 321
318:            address _mainAccount,
319:            MunchablesCommonLib.Player memory _player
320:        ) = accountManager.getPlayer(_lockRecipient);

317:        (	// @audit-issue: _player used only on line: 322
318:            address _mainAccount,
319:            MunchablesCommonLib.Player memory _player
320:        ) = accountManager.getPlayer(_lockRecipient);

375:            IERC20 token = IERC20(_tokenContract);	// @audit-issue: token used only on line: 376

329:            uint256 allowance = token.allowance(_tokenOwner, address(this));	// @audit-issue: allowance used only on line: 330

422:            IERC20 token = IERC20(_tokenContract);	// @audit-issue: token used only on line: 423
```
[287](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L287-L287), [317](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L317-L320), [317](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L317-L320), [375](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L375-L375), [329](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L329-L329), [422](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L422-L422), 


#### Recommendation

Eliminate single-use stack variables in Solidity to optimize gas consumption. Directly use the assigned value in the place of the variable. This approach saves the 3 gas typically used for the extra stack assignment, streamlining the function's execution and enhancing overall gas efficiency.

### Avoid Using State Variables Directly in `emit` for Gas Efficiency
In Solidity, emitting events is a common way to log contract activity and changes, especially for off-chain monitoring and interfacing. However, using state variables directly in `emit` statements can lead to increased gas costs. Each access to a state variable incurs gas due to storage reading operations. When these variables are used directly in `emit` statements, especially within functions that perform multiple operations, the cumulative gas cost can become significant. Instead, caching state variables in memory and using these local copies in `emit` statements can optimize gas usage.


```solidity
Path: ./src/managers/LockManager.sol

518:                    emit USDPriceUpdated(
519:                        tokenContract,
520:                        usdUpdateProposal.proposedPrice	// @audit-issue: `usdUpdateProposal` is a state variable and used on line(s): ['515', '511', '508', '509', '513']
521:                    );
```
[520](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L518-L521), 


#### Recommendation


To optimize gas efficiency, cache state variables in memory when they are used multiple times within a function, including in `emit` statements.

### Storage Layout Optimization
Storage Layout Optimization in Solidity involves arranging state variables to minimize gas costs. Since storage is expensive, combining variables into as few slots as possible and deleting unneeded variables can significantly reduce the gas needed for contract operations.

```solidity
Path: ./src/managers/LockManager.sol

16:    uint8 APPROVE_THRESHOLD = 3;	// @audit-issue: ['Current storage layout is like this: ', "However it can be optimized by 1 storage by storing variables like in order: \n\t`['configuredTokens', 'configuredTokenContracts', 'lockedTokens', 'playerSettings', 'accountManager', 'snuggeryManager', 'migrationManager', 'nftOverlord', '_usdProposalId', 'APPROVE_THRESHOLD', 'DISAPPROVE_THRESHOLD', 'lockdrop', 'usdUpdateProposal']`"]
17:    /// @notice Threshold for removing a proposal
18:    uint8 DISAPPROVE_THRESHOLD = 3;
19:    /// @notice Tokens configured on the contract
20:    mapping(address => ConfiguredToken) public configuredTokens;
21:    /// @notice Index of token contracts for easy enumerating
22:    address[] public configuredTokenContracts;
23:    /// @notice Player's currently locked tokens, can be multiple. Indexed by player then token contract
24:    mapping(address => mapping(address => LockedToken)) public lockedTokens;
25:    /// @notice Lock settings for each player
26:    mapping(address => PlayerSettings) playerSettings;
27:    /// @notice Current lockdrop start and end
28:    Lockdrop public lockdrop;
29:    /// @notice Current USD update proposal
30:    USDUpdateProposal usdUpdateProposal;
31:    /// @notice Used to make sure each approval is unique to a particular proposal
32:    uint32 private _usdProposalId;
33:
34:    /// @notice Reference to the AccountManager contract
35:    IAccountManager public accountManager;
36:
37:    /// @notice Reference to the SnuggeryManager.sol contract
38:    ISnuggeryManager public snuggeryManager;
39:
40:    /// @notice Reference to the MigrationManager contract
41:    IMigrationManager public migrationManager;
42:
43:    /// @notice Reference to the NFTOverlord contract to notify of unrevealed NFTs
44:    INFTOverlord public nftOverlord;
```
[16](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L16-L44), 


#### Recommendation

To optimize storage and reduce gas costs, rearrange the storage variables in a way that makes the most of each 32-byte storage slot.

### Don't emit events inside a loop
Emitting an event has an overhead of 375 gas, which will be incurred on every iteration of the loop. It is cheaper to emit only once after the loop has finished.

```solidity
Path: ./src/managers/LockManager.sol

518:                    emit USDPriceUpdated(	// @audit-issue
```
[518](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L518-L518), 


#### Recommendation

To optimize gas usage, avoid emitting events inside loops in Solidity. Instead, emit a single event after the loop completes, thereby saving the 375 gas overhead incurred on each iteration.

### `Internal` functions only called once can be inlined to save gas
If an internal function is only used once, there is no need to modularize it, unless the function calling it would otherwise be too long and complex. Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./src/managers/LockManager.sol

506:    function _execUSDPriceUpdate() internal {	// @audit-issue
```
[506](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L506-L506), 


#### Recommendation

Inline 'internal' functions in Solidity that are called only once to save gas. This avoids the additional gas cost of 20 to 40 units associated with extra JUMP instructions and stack operations required for separate function calls, unless the calling function becomes too complex.

### Use `Array.unsafeAccess()` to avoid repeated array length checks
When using storage arrays, solidity adds an internal lookup of the array's length (a Gcoldsload **2100 gas**) to ensure you don't read past the array's end. You can avoid this lookup by using [`Array.unsafeAccess()`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/94697be8a3f0dfcd95dfb13ffbd39b5973f5c65d/contracts/utils/Arrays.sol#L57) in cases where the length has already been checked, as is the case with the instances below

```solidity
Path: ./src/managers/LockManager.sol

253:            address tokenContract = configuredTokenContracts[i];	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

441:                configuredTokenContracts[i]	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

444:                configuredTokenContracts[i]	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

447:                configuredTokenContracts[i]	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

450:                configuredTokenContracts[i]	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

454:                configuredTokenContracts[i]	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

468:                lockedTokens[_player][configuredTokenContracts[i]].quantity >	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

470:                configuredTokens[configuredTokenContracts[i]].active	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

475:                        configuredTokens[configuredTokenContracts[i]].decimals);	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

478:                        lockedTokens[_player][configuredTokenContracts[i]]	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)

480:                        configuredTokens[configuredTokenContracts[i]]	// ('@audit-issue: Length of `configuredTokenContracts` already checked. ',)
```
[253](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L253-L253), [441](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L441-L441), [444](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L444-L444), [447](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L447-L447), [450](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L450-L450), [454](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L454-L454), [468](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L468-L468), [470](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L470-L470), [475](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L475-L475), [478](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L478-L478), [480](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L480-L480), 


#### Recommendation

Leverage `Array.unsafeAccess()` from OpenZeppelin's `Arrays` library for storage array accesses where the index is already known to be within bounds. Ensure that you have thoroughly checked the array length prior to using this method to avoid any risk of out-of-bounds errors. This approach is particularly beneficial in performance-critical sections of your code, such as loops over storage arrays where the length check is redundant. Incorporating `Array.unsafeAccess()` judiciously can lead to significant gas savings, making your contract more efficient. Always balance the use of such optimizations with the need for code safety and clarity.

### Consider pre-calculating the address of `address(this)` to save gas
Use `foundry`'s [`script.sol`](https://book.getfoundry.sh/reference/forge-std/compute-create-address) or `solady`'s [`LibRlp.sol`](https://github.com/Vectorized/solady/blob/main/src/utils/LibRLP.sol) to save the value in a constant, which will avoid having to spend gas to push the value on the stack every time it's used.

```solidity
Path: ./src/managers/LockManager.sol

329:            uint256 allowance = token.allowance(_tokenOwner, address(this));	// @audit-issue

376:            token.transferFrom(_tokenOwner, address(this), _quantity);	// @audit-issue
```
[329](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L329-L329), [376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L376-L376), 


#### Recommendation

To enhance gas efficiency, cache the contract's address by storing `address(this)` in a state variable at the point of contract deployment or initialization. Use this cached address throughout the contract instead of repeatedly calling `address(this)`. This practice reduces the gas cost associated with multiple computations of the contract's address, leading to more efficient contract execution, especially in scenarios with frequent usage of the contract's address.

### Counting down in for statements is more gas efficient
Looping downwards in Solidity is more gas efficient due to how the EVM compares variables. In a 'for' loop that counts down, the end condition is usually to compare with zero, which is cheaper than comparing with another number. As such, restructure your loops to count downwards where possible.

```solidity
Path: ./src/managers/LockManager.sol

252:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

438:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

466:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

512:            for (uint256 i; i < updateTokensLength; i++) {	// @audit-issue
```
[252](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L252-L252), [438](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L438-L438), [466](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L466-L466), [512](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L512-L512), 


#### Recommendation

Where feasible, refactor `for` loops in your Solidity contracts to count downwards. Adjust the loop initialization, condition, and iteration statements to decrement the loop variable and terminate the loop when it reaches zero. This approach can lead to gas savings, making your contract more efficient in terms of execution costs. Ensure that this refactoring aligns with the logic and requirements of your contract, and thoroughly test to confirm that the revised loop behavior matches the intended functionality.

### Use solady library where possible to save gas
The following OpenZeppelin imports have a Solady equivalent, as such they can be used to save GAS as Solady modules have been specifically designed to be as GAS efficient as possible

```solidity
Path: ./src/managers/LockManager.sol

4:import "@openzeppelin/contracts/token/ERC20/ERC20.sol";	// @audit-issue

5:import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";	// @audit-issue
```
[4](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L4-L4), [5](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L5-L5), 


#### Recommendation

Evaluate and, where appropriate, integrate Solady modules in your Solidity contracts as alternatives to similar OpenZeppelin imports. Focus on areas where gas efficiency can be significantly improved. Ensure that any replacement with Solady's modules does not compromise the security or functionality of your contracts. Conduct thorough testing and code reviews when making such substitutions to confirm compatibility and maintain the integrity of your application. Stay informed about updates and community feedback on both libraries to make informed decisions about their use in your projects.

### Consider using solady's 'FixedPointMathLib'
Using Solady's "FixedPointMathLib" for multiplication or division operations in Solidity can lead to significant gas savings. This library is designed to optimize fixed-point arithmetic operations, which are common in financial calculations involving tokens or currencies. By implementing more efficient algorithms and assembly optimizations, "FixedPointMathLib" minimizes the computational resources required for these operations. For contracts that frequently perform such calculations, integrating this library can reduce transaction costs, thereby enhancing overall performance and cost-effectiveness. However, developers must ensure compatibility with their existing codebase and thoroughly test for accuracy and expected behavior to avoid any unintended consequences.

```solidity
Path: ./src/managers/LockManager.sol

364:                numberNFTs = (quantity - remainder) / configuredToken.nftCost;	// @audit-issue

477:                    (deltaDecimal *	// @audit-issue
```
[364](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L364-L364), [477](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L477-L477), 


#### Recommendation

Consider integrating Solady's 'FixedPointMathLib' into your Solidity contracts for optimized fixed-point arithmetic operations. This library can provide substantial gas savings and enhance the performance of your contract. Before integration, evaluate how 'FixedPointMathLib' aligns with your contract’s requirements. Ensure thorough testing for accuracy and compatibility with your existing contract logic. Carefully document any changes and keep track of how these optimizations affect your contract's operations to maintain transparency and reliability in your application. Adopting 'FixedPointMathLib' should be a considered decision, balancing the benefits of gas efficiency with the need for maintaining code clarity and functionality.

### Consider using OZ EnumerateSet in place of nested mappings
Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).

A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.

OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.


```solidity
Path: ./src/managers/LockManager.sol

24:    mapping(address => mapping(address => LockedToken)) public lockedTokens;	// @audit-issue
```
[24](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L24-L24), 


#### Recommendation

Consider using OpenZeppelin's EnumerableSet library as a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays, especially when managing unique elements. This library simplifies the handling of unique data sets and allows for the enumeration of elements, a feature not natively supported in Solidity. When refactoring your contract, replace nested mappings or arrays with EnumerableSet where appropriate, ensuring both gas efficiency and enhanced functionality. Be sure to understand the use cases and limitations of EnumerableSet to apply it effectively in your contract's design and implementation.

### Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Each operation involving a `uint8` costs an extra [22-28](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed


```solidity
Path: ./src/managers/LockManager.sol

16:    uint8 APPROVE_THRESHOLD = 3;	// @audit-issue

18:    uint8 DISAPPROVE_THRESHOLD = 3;	// @audit-issue

32:    uint32 private _usdProposalId;	// @audit-issue

130:        uint8 _approve,	// @audit-issue

131:        uint8 _disapprove	// @audit-issue

263:                uint32 lastLockTime = lockedTokens[msg.sender][tokenContract]	// @audit-issue

347:        uint32 _lockDuration = playerSettings[_lockRecipient].lockDuration;	// @audit-issue
```
[16](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L16-L16), [18](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L18-L18), [32](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L32-L32), [130](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L130-L130), [131](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L131-L131), [263](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L263-L263), [347](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L347-L347), 


#### Recommendation

Minimize gas overhead by using 'uint256' or 'int256' instead of smaller integer types in Solidity contracts. The EVM operates more efficiently with 32-byte sizes. Downcast to smaller types only when necessary, as operations with smaller types like 'uint8' incur extra gas due to additional EVM operations for size adjustment.

### Refactor modifiers to call a local function
Modifiers code is copied in all instances where it's used, increasing bytecode size. If deployment gas costs are a concern for this contract, refactoring modifiers into functions can reduce bytecode size significantly at the cost of one JUMP.

```solidity
Path: ./src/managers/LockManager.sol

47:    modifier onlyConfiguredToken(address _tokenContract) {	// @audit-issue

54:    modifier onlyActiveToken(address _tokenContract) {	// @audit-issue
```
[47](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L47-L47), [54](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L54-L54), 


#### Recommendation

Evaluate your contract's use of modifiers, particularly those applied across multiple functions, to identify candidates for refactoring into functions. Convert these modifiers into external or public functions that perform the same checks or actions. Then, replace the modifier usage in function declarations with calls to these functions at the start of your functions

### Avoid Unnecessary Public Variables
Public state variables in Solidity automatically generate getter functions, increasing contract size and potentially leading to higher deployment and interaction costs. To optimize gas usage and contract efficiency, minimize the use of public variables unless external access is necessary. Instead, use internal or private visibility combined with explicit getter functions when required. This practice not only reduces contract size but also provides better control over data access and manipulation, enhancing security and readability. Prioritize lean, efficient contracts to ensure cost-effectiveness and better performance on the blockchain.

```solidity
Path: ./src/managers/LockManager.sol

7:import "../interfaces/IConfigStorage.sol";	// @audit-issue

18:    uint8 DISAPPROVE_THRESHOLD = 3;	// @audit-issue

19:    /// @notice Tokens configured on the contract	// @audit-issue

25:    /// @notice Lock settings for each player	// @audit-issue

26:    mapping(address => PlayerSettings) playerSettings;	// @audit-issue

20:    mapping(address => ConfiguredToken) public configuredTokens;	// @audit-issue

22:    address[] public configuredTokenContracts;	// @audit-issue

24:    mapping(address => mapping(address => LockedToken)) public lockedTokens;	// @audit-issue

28:    Lockdrop public lockdrop;	// @audit-issue

35:    IAccountManager public accountManager;	// @audit-issue

38:    ISnuggeryManager public snuggeryManager;	// @audit-issue

41:    IMigrationManager public migrationManager;	// @audit-issue

44:    INFTOverlord public nftOverlord;	// @audit-issue
```
[7](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L7-L7), [18](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L18-L18), [19](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L19-L19), [25](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L25-L25), [26](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L26-L26), [20](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L20-L20), [22](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L22-L22), [24](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L24-L24), [28](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L28-L28), [35](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L35-L35), [38](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L38-L38), [41](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L41-L41), [44](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L44-L44), 


#### Recommendation

Avoid creating explicit getter functions for 'public' state variables in Solidity. The compiler automatically generates getters for such variables, making additional functions redundant. This practice helps reduce contract size, lowers deployment costs, and simplifies maintenance and understanding of the contract.

### Use `do while` loops intead of for loops
A `do while` loop will cost less gas since the condition is not being checked for the first iteration.
```solidity
uint256 i = 1;
do {                   
    param2 += i;
    i++;
}
while (i < 50);
``` 
is better than
```solidity
for(uint256 i = 1; i< 50; i++){
    param1 += i;
}
```


```solidity
Path: ./src/managers/LockManager.sol

252:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

438:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

466:        for (uint256 i; i < configuredTokensLength; i++) {	// @audit-issue

512:            for (uint256 i; i < updateTokensLength; i++) {	// @audit-issue
```
[252](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L252-L252), [438](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L438-L438), [466](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L466-L466), [512](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L512-L512), 


#### Recommendation

Where appropriate, consider using a `do while` loop instead of a `for` loop in your Solidity contracts. This is especially beneficial when the first iteration of the loop does not require a condition check. Refactor your loop logic to fit the `do while` structure for more gas-efficient execution. However, ensure that the loop's logic and termination conditions are correctly implemented to avoid infinite loops or other logical errors. Always balance gas efficiency with code readability and the specific requirements of your contract's logic.

### Using XOR (^) and AND (&) bitwise equivalents for gas optimizations
Given 4 variables a, b, c and d represented as such:
```
0 0 0 0 0 1 1 0 <- a
0 1 1 0 0 1 1 0 <- b
0 0 0 0 0 0 0 0 <- c
1 1 1 1 1 1 1 1 <- d
```
To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.


```solidity
Path: ./src/managers/LockManager.sol

48:        if (configuredTokens[_tokenContract].nftCost == 0)	// @audit-issue

119:        if (_tokenData.nftCost == 0) revert NFTCostInvalidError();	// @audit-issue

120:        if (configuredTokens[_tokenContract].nftCost == 0) {	// @audit-issue

159:        if (_contracts.length == 0) revert ProposalInvalidContractsError();	// @audit-issue

191:        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();	// @audit-issue

192:        if (usdUpdateProposal.proposer == msg.sender)	// @audit-issue

194:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)	// @audit-issue

224:        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();	// @audit-issue

225:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)	// @audit-issue

227:        if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)	// @audit-issue

322:        if (_player.registrationDate == 0) revert AccountNotRegisteredError();	// @audit-issue

324:        if (_tokenContract == address(0)) {	// @audit-issue

349:        if (_lockDuration == 0) {	// @audit-issue

419:        if (_tokenContract == address(0)) {	// @audit-issue
```
[48](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L48-L48), [119](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L119-L119), [120](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L120-L120), [159](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L159-L159), [191](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L191-L191), [192](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L192-L192), [194](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L194-L194), [224](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L224-L224), [225](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L225-L225), [227](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L227-L227), [322](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L322-L322), [324](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L324-L324), [349](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L349-L349), [419](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L419-L419), 


#### Recommendation

Review your Solidity contracts to identify any computations that are performed multiple times with the same inputs. Cache the results of these computations in local variables and reuse them within the function or across function calls if the state remains unchanged.

### The result of a function call should be cached rather than re-calling the function
The function calls in solidity are expensive. If the same result of the same function calls are to be used several times, the result should be cached to reduce the gas consumption of repeated calls.        

```solidity
Path: ./src/managers/LockManager.sol

328:            IERC20 token = IERC20(_tokenContract);	// @audit-issue: Function call `IERC20` is called multiple times at lines [375].
```
[328](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L328-L328), 


#### Recommendation

Cache the result of function calls in Solidity instead of making repeated calls to the same function. This practice significantly reduces gas consumption by minimizing costly function call operations.

### Avoid updating storage when the value hasn't changed
Manipulating storage in solidity is gas-intensive. It can be optimized by avoiding unnecessary storage updates when the new value equals the existing value. If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas), potentially at the expense of a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas).

```solidity
Path: ./src/managers/LockManager.sol

109:        lockdrop = _lockdropData;	// @audit-issue

124:        configuredTokens[_tokenContract] = _tokenData;	// @audit-issue

135:        APPROVE_THRESHOLD = _approve;	// @audit-issue

136:        DISAPPROVE_THRESHOLD = _disapprove;	// @audit-issue

168:        usdUpdateProposal.proposedPrice = _price;	// @audit-issue

169:        usdUpdateProposal.contracts = _contracts;	// @audit-issue
```
[109](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L109-L109), [124](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L124-L124), [135](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L135-L135), [136](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L136-L136), [168](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L168-L168), [169](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L169-L169), 


#### Recommendation

Optimize gas usage by avoiding storage updates in Solidity when the new value is the same as the existing one. This prevents unnecessary gas expenditure from storage resets, balancing the cost against cold or warm storage access as needed.

### Functions guaranteed to `revert` when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
Path: ./src/managers/LockManager.sol

85:    function configUpdated() external override onlyConfigStorage {	// @audit-issue

98:    function configureLockdrop(	// @audit-issue

115:    function configureToken(	// @audit-issue

129:    function setUSDThresholds(	// @audit-issue

142:    function proposeUSDPrice(	// @audit-issue

177:    function approveUSDPrice(	// @audit-issue

210:    function disapproveUSDPrice(	// @audit-issue
```
[85](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L85-L85), [98](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L98-L98), [115](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L115-L115), [129](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L129-L129), [142](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L142-L142), [177](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L177-L177), [210](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L210-L210), 


#### Recommendation

Mark functions with access restrictions like 'onlyOwner' as 'payable' in Solidity. This reduces gas costs for legitimate callers by removing the compiler's checks for incoming payments, as the function will revert for unauthorized users anyway.

### `x += y` costs more gas than `x = x + y` for stack variables
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./src/managers/LockManager.sol

380:        lockedToken.quantity += _quantity;	// @audit-issue

416:        lockedToken.quantity -= _quantity;	// @audit-issue

476:                lockedWeighted +=	// @audit-issue
```
[380](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L380-L380), [416](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L416-L416), [476](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L476-L476), 


#### Recommendation

Prefer using 'x = x + y' over 'x += y' for state variable assignments in Solidity to save gas. The latter incurs extra costs due to additional JUMP instructions and stack operations associated with non-inlined function calls.

### Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

```solidity
Path: ./src/managers/LockManager.sol

115:    function configureToken(
116:        address _tokenContract,
117:        ConfiguredToken memory _tokenData	// @audit-issue
118:    ) external onlyAdmin {
```
[117](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L115-L118), 


#### Recommendation

To optimize gas usage in your Solidity functions, mark data types as `calldata` instead of `memory` wherever applicable. This prevents unnecessary data loading into memory. Use `calldata` for function arguments that do not require changes within the function, except when passing them into another function that explicitly requires `memory` storage.

### Nesting `if` statements that uses `&&` saves gas
In Solidity, the way conditional checks are structured can impact the gas consumption of a transaction. When conditions are combined using `&&` within an `if` statement, Solidity short-circuits the evaluation, meaning that if the first condition is `false`, the subsequent conditions won't be evaluated. This behavior can lead to gas savings compared to using separate nested `if` statements because not all conditions might need to be checked every time. By efficiently structuring these conditional checks, contracts can optimize the gas required for execution, leading to reduced costs for users.


```solidity
Path: ./src/managers/LockManager.sol

353:            lockdrop.start <= uint32(block.timestamp) &&	// @audit-issue
354:            lockdrop.end >= uint32(block.timestamp)

468:                lockedTokens[_player][configuredTokenContracts[i]].quantity >	// @audit-issue
469:                0 &&
470:                configuredTokens[configuredTokenContracts[i]].active

508:            usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD &&	// @audit-issue
509:            usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD
```
[353](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L353-L354), [468](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L468-L470), [508](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L508-L509), 


#### Recommendation


When multiple conditions need to be checked successively, try to combine them in a single `if` statement using `&&` instead of nesting separate `if` statements. This will leverage short-circuit evaluation for potential gas savings.


### Use `!= 0` Instead of `> 0` for Unsigned Integer Comparison
In Solidity, unsigned integers (e.g., `uint256`, `uint8`, etc.) represent non-negative whole numbers, ranging from 0 to a maximum value determined by their bit size. When performing comparisons on these numbers, especially to check if they are non-zero, developers have options. A common practice is to compare against zero using the `>` operator, as in `value > 0`. However, given the nature of unsigned integers, a more straightforward and slightly gas-efficient comparison is to use the `!=` operator, as in `value != 0`.

The primary rationale is that the `!=` comparison directly checks for non-equality, whereas the `>` comparison checks if one value is strictly greater than another. For unsigned integers, where negative values don't exist, the `!= 0` check is both semantically clearer and potentially more efficient at the EVM bytecode level.


```solidity
Path: ./src/managers/LockManager.sol

254:            if (lockedTokens[msg.sender][tokenContract].quantity > 0) {	// @audit-issue

468:                lockedTokens[_player][configuredTokenContracts[i]].quantity >	// @audit-issue
```
[254](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L254-L254), [468](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L468-L468), 


#### Recommendation

Use '!= 0' instead of '> 0' for comparing unsigned integers in Solidity. This is semantically clearer and slightly more gas-efficient, as it directly checks for non-zero values without considering unnecessary relational comparisons.

### Constructor Can Be Marked As Payable
`payable` functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided.

A `constructor` can safely be marked as `payable`, since only the deployer would be able to pass funds, and the project itself would not pass any funds.



```solidity
Path: ./src/managers/LockManager.sol

60:    constructor(address _configStorage) {	// @audit-issue
```
[60](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L60-L60), 


#### Recommendation

Mark constructors as 'payable' in Solidity contracts to reduce gas costs, as this eliminates the need for the compiler to add checks against incoming payments. This is safe because only the deployer can send funds during contract creation, and typically no funds are sent at this stage.

### Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).


```solidity
Path: ./src/managers/LockManager.sol

14:contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {	// @audit-issue
```
[14](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L14-L14), 


#### Recommendation

Optimize gas usage by renaming 'public'/'external' functions and 'public' member variables in Solidity. Aim for shorter and more efficient names, especially for frequently called functions. This can save gas during deployment and reduce gas costs per call due to lower method ID sorting positions.

### Consider activating `via-ir` for deploying
The IR-based code generator was developed to make code generation more performant by enabling optimization passes that can be applied across functions.

It is possible to activate the IR-based code generator through the command line by using the flag `--via-ir`or by including the option `{"viaIR": true}`.

Keep in mind that compiling with this option may take longer. However, you can simply test it before deploying your code. If you find that it provides better performance, you can add the `--via-ir` flag to your deploy command.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L1), 


#### Recommendation

Consider activating `via-ir`.

### Optimize Deployment Size by Fine-tuning IPFS Hash
The Solidity compiler appends 53 bytes of metadata to the smart contract code, incurring an extra cost of 10,600 gas. This additional expense arises from 200 gas per bytecode, plus calldata cost, which amounts to 16 gas for non-zero bytes and 4 gas for zero bytes. This results in a maximum of 848 extra gas in calldata cost.

Reducing this cost is crucial for the following reasons:

The metadata's 53-byte addition leads to a deployment cost increase of 10,600 gas. It can also result in an additional calldata cost of up to 848 gas. Ways to Minimize Gas Consumption:

Employ the `--no-cbor-metadata` compiler option to exclude metadata. Be cautious as this might impact contract verification. Search for code comments that yield an IPFS hash with more zeros, thereby reducing calldata costs.

```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L1), 


#### Recommendation

To optimize deployment size and reduce associated costs, consider the following strategies:
1. **Exclude Metadata with Compiler Option**: Use the `solc` compiler’s `--metadata-hash none` or `--no-cbor-metadata` option to prevent the inclusion of metadata in the compiled bytecode. This action reduces the bytecode size, thus lowering deployment gas costs. However, exercise caution with this approach, as it might affect the ability to verify the contract on platforms like Etherscan.

2. **Optimize IPFS Hash for More Zeros**: If excluding metadata is not desirable, another approach involves optimizing code comments or elements that influence the metadata hash generation to achieve an IPFS hash with a higher proportion of zeros. Since calldata costs are lower for zero bytes, a metadata hash with more zeros can reduce the calldata costs associated with contract interactions.

Example for excluding metadata:
```bash
solc --metadata-hash none YourContract.sol
```


### Assembly: Use scratch space for building calldata
If an external call's calldata can fit into two or fewer words, use the scratch space to build the calldata, rather than allowing Solidity to do a memory expansion.

```solidity
Path: ./src/managers/LockManager.sol

67:            configStorage.getAddress(StorageKey.AccountManager)	// @audit-issue

71:            configStorage.getAddress(StorageKey.MigrationManager)	// @audit-issue

75:            configStorage.getAddress(StorageKey.SnuggeryManager)	// @audit-issue

79:            configStorage.getAddress(StorageKey.NFTOverlord)	// @audit-issue

82:        super.__BaseBlastManager_reconfigure();	// @audit-issue

122:            configuredTokenContracts.push(_tokenContract);	// @audit-issue

246:        if (_duration > configStorage.getUint(StorageKey.MaxLockDuration))	// @audit-issue

320:        ) = accountManager.getPlayer(_lockRecipient);	// @audit-issue

329:            uint256 allowance = token.allowance(_tokenOwner, address(this));	// @audit-issue

341:        accountManager.forceHarvest(_lockRecipient);	// @audit-issue

359:                uint32(configStorage.getUint(StorageKey.MaxLockDuration))	// @audit-issue

369:                nftOverlord.addReveal(_lockRecipient, uint16(numberNFTs));	// @audit-issue

414:        accountManager.forceHarvest(msg.sender);	// @audit-issue

420:            payable(msg.sender).transfer(_quantity);	// @audit-issue

423:            token.transfer(msg.sender, _quantity);	// @audit-issue
```
[67](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L67-L67), [71](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L71-L71), [75](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L75-L75), [79](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L79-L79), [82](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L82-L82), [122](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L122-L122), [246](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L246-L246), [320](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L320-L320), [329](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L329-L329), [341](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L341-L341), [359](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L359-L359), [369](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L369-L369), [414](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L414-L414), [420](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L420-L420), [423](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L423-L423), 


#### Recommendation

Review your smart contracts to identify and remove `block.number` and `block.timestamp` from the parameters of emitted events. Instead of manually adding these fields, rely on the Ethereum blockchain's inherent inclusion of this information within the block context. Simplify your event definitions to include only the essential data specific to the event's purpose, excluding universally available block metadata.

### Use assembly to check for `address(0)`
In Solidity, it's a common practice to check whether an Ethereum address variable is set to the zero address (`address(0)`) to handle various scenarios, such as token transfers or contract interactions. Typically, this check is performed using a conditional statement like `if (addressVariable == address(0))`.

However, using this approach in high-frequency or gas-sensitive operations can lead to unnecessary gas costs. A more gas-efficient alternative is to use inline assembly to perform the zero address check, which can significantly reduce gas consumption, especially in loops or complex contract logic.

By utilizing inline assembly for this specific check, you can optimize gas usage and make your Solidity code more efficient.

```solidity
Path: ./src/managers/LockManager.sol

133:        if (usdUpdateProposal.proposer != address(0))	// @audit-issue

157:        if (usdUpdateProposal.proposer != address(0))	// @audit-issue

191:        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();	// @audit-issue

224:        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();	// @audit-issue

289:        if (_onBehalfOf != address(0)) {	// @audit-issue

324:        if (_tokenContract == address(0)) {	// @audit-issue

374:        if (_tokenContract != address(0)) {	// @audit-issue

419:        if (_tokenContract == address(0)) {	// @audit-issue
```
[133](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L133-L133), [157](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L157-L157), [191](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L191-L191), [224](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L224-L224), [289](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L289-L289), [324](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L324-L324), [374](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L374-L374), [419](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L419-L419), 


#### Recommendation

To optimize gas usage in your Solidity code, consider using inline assembly for checking `address(0)`. This approach can significantly reduce gas costs, especially in high-frequency or gas-sensitive operations, leading to more efficient contract execution.

### Use assembly to check for `0`
Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```solidity
Path: ./src/managers/LockManager.sol

119:        if (_tokenData.nftCost == 0) revert NFTCostInvalidError();	// @audit-issue

120:        if (configuredTokens[_tokenContract].nftCost == 0) {	// @audit-issue

159:        if (_contracts.length == 0) revert ProposalInvalidContractsError();	// @audit-issue

254:            if (lockedTokens[msg.sender][tokenContract].quantity > 0) {	// @audit-issue

322:        if (_player.registrationDate == 0) revert AccountNotRegisteredError();	// @audit-issue

327:            if (msg.value != 0) revert InvalidMessageValueError();	// @audit-issue

349:        if (_lockDuration == 0) {	// @audit-issue

468:                lockedTokens[_player][configuredTokenContracts[i]].quantity >	// @audit-issue

514:                if (configuredTokens[tokenContract].nftCost != 0) {	// @audit-issue
```
[119](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L119-L119), [120](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L120-L120), [159](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L159-L159), [254](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L254-L254), [322](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L322-L322), [327](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L327-L327), [349](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L349-L349), [468](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L468-L468), [514](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L514-L514), 


#### Recommendation

To optimize gas usage in your Solidity code, consider using inline assembly for checking `0`. This approach can significantly reduce gas costs, especially in high-frequency or gas-sensitive operations, leading to more efficient contract execution.

### Use assembly to emit an `event`
To efficiently emit events, it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion.

However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

A good example of such practice can be seen in [Solady's](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167) codebase.


```solidity
Path: ./src/managers/LockManager.sol

111:        emit LockDropConfigured(_lockdropData);	// @audit-issue

126:        emit TokenConfigured(_tokenContract, _tokenData);	// @audit-issue

138:        emit USDThresholdUpdated(_approve, _disapprove);	// @audit-issue

173:        emit ProposedUSDPrice(msg.sender, _price);	// @audit-issue

206:        emit ApprovedUSDPrice(msg.sender);	// @audit-issue

235:        emit DisapprovedUSDPrice(msg.sender);	// @audit-issue

240:            emit RemovedUSDProposal();	// @audit-issue

271:        emit LockDuration(msg.sender, _duration);	// @audit-issue

389:        emit Locked(	// @audit-issue

426:        emit Unlocked(msg.sender, _tokenContract, _quantity);	// @audit-issue

518:                    emit USDPriceUpdated(	// @audit-issue
```
[111](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L111-L111), [126](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L126-L126), [138](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L138-L138), [173](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L173-L173), [206](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L206-L206), [235](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L235-L235), [240](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L240-L240), [271](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L271-L271), [389](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L389-L389), [426](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L426-L426), [518](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L518-L518), 


#### Recommendation

To optimize event emission in your Solidity code, consider using assembly with scratch space and the free memory pointer. This approach can help reduce gas costs by avoiding memory expansion expenses. However, it's crucial to ensure safe optimization by caching and restoring the free memory pointer, as demonstrated in examples like Solady's codebase.

### Use assembly to validate `msg.sender`
We can use assembly to efficiently validate `msg.sender` with the least amount of opcodes necessary. For more details check the following report [Here](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender)


```solidity
Path: ./src/managers/LockManager.sol

192:        if (usdUpdateProposal.proposer == msg.sender)	// @audit-issue

361:            if (msg.sender != address(migrationManager)) {	// @audit-issue
```
[192](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L192-L192), [361](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L361-L361), 


#### Recommendation

To optimize the validation of `msg.sender` in your Solidity code, consider using assembly to achieve this with the minimum number of opcodes required. You can refer to the detailed report [Here](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender) for more insights and examples on efficient implementation.
