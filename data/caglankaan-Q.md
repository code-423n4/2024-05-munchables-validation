### Return values of `transfer()`/`transferFrom()` not checked
Not all `ERC20` implementations `revert()` when there's a failure in `transfer()` or `transferFrom()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually transfer anything.

```solidity
Path: ./src/managers/LockManager.sol

376:            token.transferFrom(_tokenOwner, address(this), _quantity);	// @audit-issue

423:            token.transfer(msg.sender, _quantity);	// @audit-issue
```
[376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L376-L376), [423](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L423-L423), 


#### Recommendation

To ensure the reliability and security of token transfers in your smart contract, it's crucial to check the return values of the `transfer()` and `transferFrom()` functions. These functions often return a boolean value indicating the success or failure of the transfer operation. By checking this return value, you can accurately determine whether the transfer was successful and handle any potential errors or failures accordingly. Failing to check the return value may lead to unintended and unhandled transfer failures, which could have security and usability implications.

### Centralization risk for privileged functions
Contracts with privileged functions need owner/admin to be trusted not to perform malicious updates or drain funds. This may also cause a single point failure.


```solidity
Path: ./src/managers/LockManager.sol

85:    function configUpdated() external override onlyConfigStorage {	// @audit-issue

98:    function configureLockdrop(
99:        Lockdrop calldata _lockdropData
100:    ) external onlyAdmin {	// @audit-issue

115:    function configureToken(
116:        address _tokenContract,
117:        ConfiguredToken memory _tokenData
118:    ) external onlyAdmin {	// @audit-issue

129:    function setUSDThresholds(
130:        uint8 _approve,
131:        uint8 _disapprove
132:    ) external onlyAdmin {	// @audit-issue

142:    function proposeUSDPrice(
143:        uint256 _price,
144:        address[] calldata _contracts
145:    )
146:        external
147:        onlyOneOfRoles(	// @audit-issue
148:            [
149:                Role.PriceFeed_1,
150:                Role.PriceFeed_2,
151:                Role.PriceFeed_3,
152:                Role.PriceFeed_4,
153:                Role.PriceFeed_5
154:            ]
155:        )

177:    function approveUSDPrice(
178:        uint256 _price
179:    )
180:        external
181:        onlyOneOfRoles(	// @audit-issue
182:            [
183:                Role.PriceFeed_1,
184:                Role.PriceFeed_2,
185:                Role.PriceFeed_3,
186:                Role.PriceFeed_4,
187:                Role.PriceFeed_5
188:            ]
189:        )

210:    function disapproveUSDPrice(
211:        uint256 _price
212:    )
213:        external
214:        onlyOneOfRoles(	// @audit-issue
215:            [
216:                Role.PriceFeed_1,
217:                Role.PriceFeed_2,
218:                Role.PriceFeed_3,
219:                Role.PriceFeed_4,
220:                Role.PriceFeed_5
221:            ]
222:        )

275:    function lockOnBehalf(
276:        address _tokenContract,
277:        uint256 _quantity,
278:        address _onBehalfOf
279:    )
280:        external
281:        payable
282:        notPaused
283:        onlyActiveToken(_tokenContract)	// @audit-issue

275:    function lockOnBehalf(
276:        address _tokenContract,
277:        uint256 _quantity,
278:        address _onBehalfOf
279:    )
280:        external
281:        payable
282:        notPaused
283:        onlyActiveToken(_tokenContract)
284:        onlyConfiguredToken(_tokenContract)	// @audit-issue

297:    function lock(
298:        address _tokenContract,
299:        uint256 _quantity
300:    )
301:        external
302:        payable
303:        notPaused
304:        onlyActiveToken(_tokenContract)	// @audit-issue

297:    function lock(
298:        address _tokenContract,
299:        uint256 _quantity
300:    )
301:        external
302:        payable
303:        notPaused
304:        onlyActiveToken(_tokenContract)
305:        onlyConfiguredToken(_tokenContract)	// @audit-issue
```
[85](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L85-L85), [100](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L98-L100), [118](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L115-L118), [132](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L129-L132), [147](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L142-L155), [181](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L177-L189), [214](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L210-L222), [283](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L275-L283), [284](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L275-L284), [304](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L297-L304), [305](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L297-L305), 


#### Recommendation

To mitigate centralization risks associated with privileged functions, consider implementing a multi-signature or decentralized governance mechanism. Instead of relying solely on a single owner/administrator, distribute control and decision-making authority among multiple parties or stakeholders. This approach enhances security, reduces the risk of malicious actions by a single entity, and prevents single points of failure. Explore governance solutions and smart contract frameworks that support decentralized control and decision-making to enhance the trustworthiness and resilience of your contract.

### Code does not follow the best practice of check-effects-interactions pattern
Code should follow the best-practice of [CEI](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.

```solidity
Path: ./src/managers/LockManager.sol

376:            token.transferFrom(_tokenOwner, address(this), _quantity);
377:        }
378:
379:        lockedToken.remainder = remainder;
380:        lockedToken.quantity += _quantity;
381:        lockedToken.lastLockTime = uint32(block.timestamp);
382:        lockedToken.unlockTime =
383:            uint32(block.timestamp) +
384:            uint32(_lockDuration);
385:
386:        // set their lock duration in playerSettings
387:        playerSettings[_lockRecipient].lockDuration = _lockDuration;	// @audit-issue
```
[387](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L376-L387), 


#### Recommendation

To enhance the security of your smart contract and prevent reentrancy attacks, it's crucial to adhere to the check-effects-interactions pattern. Restructure your contract's functions to follow this pattern by first performing condition checks (checks), then updating the contract's state (effects), and finally, executing any external calls or interactions (interactions). This sequential approach reduces the risk of reentrancy vulnerabilities by ensuring that state changes are isolated from external interactions and condition checks are made before any state modifications. Review and refactor your contract code to align with this best practice for improved security.

### Missing checks for `address(0)` in constructor/initializers
In Solidity, the Ethereum address `0x0000000000000000000000000000000000000000` is known as the "zero address". This address has significance because it's the default value for uninitialized address variables and is often used to represent an invalid or non-existent address. The "
Missing zero address control" issue arises when a Solidity smart contract does not properly check or prevent interactions with the zero address, leading to unintended behavior.
For instance, a contract might allow tokens to be sent to the zero address without any checks, which essentially burns those tokens as they become irretrievable. While sometimes this is intentional, without proper control or checks, accidental transfers could occur.    
        

```solidity
Path: ./src/managers/LockManager.sol

61:        __BaseConfigStorage_setConfigStorage(_configStorage);	// @audit-issue
```
[61](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L61-L61), 


#### Recommendation

It is strongly recommended to implement checks to prevent the zero address from being set during the initialization of contracts. This can be achieved by adding require statements that ensure address parameters are not the zero address. 

### Unsafe downcast may overflow
When a type is downcast to a smaller type, the higher order bits are discarded, resulting in the application of a modulo operation to the original value.

If the downcasted value is large enough, this may result in an overflow that will not revert.


```solidity
Path: ./src/managers/LockManager.sol

369:                nftOverlord.addReveal(_lockRecipient, uint16(numberNFTs));	// @audit-issue: Variable `numberNFTs` is type `uint256` and it is downcasted to `uint16`
```
[369](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L369-L369), 


#### Recommendation

When downcasting numbers to `addresses` in Solidity, be cautious of potential collisions. Downcasting truncates the upper bytes of the number, which can lead to multiple values mapping to the same `address`. To avoid collisions, consider using a more robust mapping strategy or additional checks to ensure unique mappings.

### Use Of `transfer` or `send` Instead Of `call` To Send Native Assets
The use of `transfer()` in the contracts may have unintended outcomes on the native asset being sent to the receiver. The transaction will fail when:

- The receiver address is a smart contract that does not implement a payable function.
- The receiver address is a smart contract that implements a payable fallback function which uses more than 2300 gas units.
- The receiver address is a smart contract that implements a payable fallback function that needs less than 2300 gas units but is called through proxy, raising the call's gas usage above 2300.
- Additionally, using a gas value higher than 2300 might be mandatory for some multi-signature wallets.




```solidity
Path: ./src/managers/LockManager.sol

420:            payable(msg.sender).transfer(_quantity);	// @audit-issue
```
[420](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L420-L420), 


#### Recommendation


To transfer, a native asset is recommended to either use:

- `<receiver>.call.value(amount)`
- [OpenZeppelin Address.sendValue()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v2.5.1/contracts/utils/Address.sol#L63-L69)

In both cases, care must be taken not to create reentrancy vulnerabilities. Consider using ReentrancyGuard or the [Check-Effects-Interactions pattern](https://docs.soliditylang.org/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern).




### Int casting `block.timestamp` can reduce the lifespan of a contract
In Solidity, `block.timestamp` returns the Unix timestamp of the current block, which is a continuously increasing value. Casting `block.timestamp` to a smaller integer type, like `uint32`, can limit the maximum value it can hold. As time progresses and the Unix timestamp exceeds this maximum value, the casted value will overflow, leading to incorrect and potentially harmful behavior in the contract. This issue can significantly reduce the functional lifespan of a contract, as it becomes unreliable and potentially insecure once the timestamp exceeds the capacity of the casted integer type.

```solidity
Path: ./src/managers/LockManager.sol

104:                uint32(block.timestamp)	// @audit-issue

166:        usdUpdateProposal.proposedDate = uint32(block.timestamp);	// @audit-issue

257:                    uint32(block.timestamp) + uint32(_duration) <	// @audit-issue

353:            lockdrop.start <= uint32(block.timestamp) &&	// @audit-issue

354:            lockdrop.end >= uint32(block.timestamp)	// @audit-issue

381:        lockedToken.lastLockTime = uint32(block.timestamp);	// @audit-issue

383:            uint32(block.timestamp) +	// @audit-issue

410:        if (lockedToken.unlockTime > uint32(block.timestamp))	// @audit-issue
```
[104](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L104-L104), [166](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L166-L166), [257](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L257-L257), [353](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L353-L353), [354](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L354-L354), [381](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L381-L381), [383](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L383-L383), [410](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L410-L410), 


#### Recommendation

Avoid casting `block.timestamp` to smaller integer types in your Solidity contracts. Use `uint256` to store timestamp values to accommodate the increasing nature of Unix timestamps and ensure the contract remains functional in the long term. Regularly review and update your contracts to remove any unnecessary casting of `block.timestamp` that could lead to overflows and reduce the contract's lifespan. By doing so, you maintain the contract's reliability and integrity well into the future.

### Functions calling contracts/addresses with transfer hooks are missing reentrancy guards
Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to [read-only reentrancies](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/) with no way to protect against it, except by block-listing the whole protocol.

```solidity
Path: ./src/managers/LockManager.sol

376:            token.transferFrom(_tokenOwner, address(this), _quantity);	// @audit-issue
```
[376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L376-L376), 


#### Recommendation

To ensure the security of your protocol and protect users from potential vulnerabilities, consider implementing reentrancy guards in functions that call contracts or addresses with transfer hooks. Even if your function follows the check-effects-interaction pattern, using reentrancy guards is essential to prevent read-only reentrancy attacks, as demonstrated in incidents like the [Curve LP Oracle Manipulation](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/). Implementing reentrancy guards adds an extra layer of protection and safeguards your protocol against such attacks.

### Consider using OpenZeppelin’s `SafeCast` library to prevent unexpected overflows when casting from various type int/uint values
In Solidity, when casting from `int` to `uint` or vice versa, there is a risk of unexpected overflow or underflow, especially when dealing with large values. To mitigate this risk and ensure safe type conversions, you can consider using OpenZeppelin’s `SafeCast` library. This library provides functions that check for overflow/underflow conditions before performing the cast, helping you prevent potential issues related to type conversion in your smart contracts.

```solidity
Path: ./src/managers/LockManager.sol

104:                uint32(block.timestamp)	// @audit-issue

166:        usdUpdateProposal.proposedDate = uint32(block.timestamp);	// @audit-issue

249:        playerSettings[msg.sender].lockDuration = uint32(_duration);	// @audit-issue

257:                    uint32(block.timestamp) + uint32(_duration) <	// @audit-issue

267:                    uint32(_duration);	// @audit-issue

353:            lockdrop.start <= uint32(block.timestamp) &&	// @audit-issue

354:            lockdrop.end >= uint32(block.timestamp)	// @audit-issue

359:                uint32(configStorage.getUint(StorageKey.MaxLockDuration))	// @audit-issue

369:                nftOverlord.addReveal(_lockRecipient, uint16(numberNFTs));	// @audit-issue

381:        lockedToken.lastLockTime = uint32(block.timestamp);	// @audit-issue

383:            uint32(block.timestamp) +	// @audit-issue

384:            uint32(_lockDuration);	// @audit-issue

410:        if (lockedToken.unlockTime > uint32(block.timestamp))	// @audit-issue
```
[104](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L104-L104), [166](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L166-L166), [249](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L249-L249), [257](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L257-L257), [267](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L267-L267), [353](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L353-L353), [354](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L354-L354), [359](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L359-L359), [369](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L369-L369), [381](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L381-L381), [383](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L383-L383), [384](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L384-L384), [410](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L410-L410), 


#### Recommendation

To enhance the safety and reliability of your Solidity smart contracts, it is advisable to utilize OpenZeppelin’s `SafeCast` library when casting between `int` and `uint` types. Incorporating this library into your codebase will help prevent unexpected overflows and underflows during type conversion, reducing the risk of vulnerabilities and ensuring secure contract execution.

### Revert on transfer to the zero address
It's good practice to revert a token transfer transaction if the recipient's address is the zero address. This can prevent unintentional transfers to the zero address due to accidental operations or programming errors. Many token contracts implement such a safeguard, such as [OpenZeppelin - ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC20/ERC20.sol#L232), [OpenZeppelin - ERC721](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC721/ERC721.sol#L142).

```solidity
Path: ./src/managers/LockManager.sol

423:            token.transfer(msg.sender, _quantity);	// @audit-issue
```
[423](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L423-L423), 


#### Recommendation

To enhance the security and reliability of your token contracts, it's advisable to implement a safeguard that reverts token transfer transactions if the recipient's address is the zero address. This practice helps prevent unintentional transfers to the zero address, reducing the risk of fund loss due to accidental operations or programming errors. Many token contracts, including [OpenZeppelin's ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC20/ERC20.sol#L232) and [ERC721](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC721/ERC721.sol#L142), incorporate this safeguard for added security.

### State variables not limited to reasonable values
Consider adding appropriate minimum/maximum value checks to ensure that the following state variables can never be used to excessively harm users, including via griefing.

```solidity
Path: ./src/managers/LockManager.sol

135:        APPROVE_THRESHOLD = _approve;	// @audit-issue

136:        DISAPPROVE_THRESHOLD = _disapprove;	// @audit-issue

168:        usdUpdateProposal.proposedPrice = _price;	// @audit-issue
```
[135](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L135-L135), [136](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L136-L136), [168](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L168-L168), 


#### Recommendation


Implement validation checks for state variables to enforce minimum and maximum value limits. This can be achieved by adding modifiers or require statements in Solidity functions that modify these state variables. Ensure that these limits are reasonable and reflect the intended use of the contract. Additionally, consider implementing a mechanism to update these limits through a governance process or a trusted role, if applicable, to maintain flexibility and adaptability of the contract over time.

### External calls in an unbounded loop can result in a DoS
Consider limiting the number of iterations in loops that make external calls, as just a single one of them failing will result in a revert.

```solidity
Path: ./src/managers/LockManager.sol

520:                        usdUpdateProposal.proposedPrice	// @audit-issue
```
[520](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L520-L520), 


#### Recommendation

To mitigate the risk of Denial-of-Service (DoS) attacks in your Solidity code, it's important to limit the number of iterations in loops that involve external calls. A single failed external call in an unbounded loop can lead to a revert, causing disruptions in contract execution. Consider implementing safeguards, such as setting a maximum loop iteration count or employing strategies like batch processing, to reduce the impact of potential external call failures.

### Critical functions should have a timelock
Critical functions, especially those affecting protocol parameters or user funds, are potential points of failure or exploitation. To mitigate risks, incorporating a timelock on such functions can be beneficial. A timelock requires a waiting period between the time an action is initiated and when it's executed, giving stakeholders time to react, potentially vetoing malicious or erroneous changes. To implement, integrate a smart contract like OpenZeppelin's `TimelockController` or build a custom mechanism. This ensures governance decisions or administrative changes are transparent and allows for community or multi-signature interventions, enhancing protocol security and trustworthiness.

```solidity
Path: ./src/managers/LockManager.sol

115:    function configureToken(
116:        address _tokenContract,
117:        ConfiguredToken memory _tokenData
118:    ) external onlyAdmin {
119:        if (_tokenData.nftCost == 0) revert NFTCostInvalidError();
120:        if (configuredTokens[_tokenContract].nftCost == 0) {
121:            // new token
122:            configuredTokenContracts.push(_tokenContract);
123:        }
124:        configuredTokens[_tokenContract] = _tokenData;	// @audit-issue

129:    function setUSDThresholds(
130:        uint8 _approve,
131:        uint8 _disapprove
132:    ) external onlyAdmin {
133:        if (usdUpdateProposal.proposer != address(0))
134:            revert ProposalInProgressError();
135:        APPROVE_THRESHOLD = _approve;	// @audit-issue

129:    function setUSDThresholds(
130:        uint8 _approve,
131:        uint8 _disapprove
132:    ) external onlyAdmin {
133:        if (usdUpdateProposal.proposer != address(0))
134:            revert ProposalInProgressError();
135:        APPROVE_THRESHOLD = _approve;
136:        DISAPPROVE_THRESHOLD = _disapprove;	// @audit-issue
```
[124](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L115-L124), [135](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L129-L135), [136](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L129-L136), 


#### Recommendation

Integrate a timelock mechanism into your Solidity contracts for critical functions, especially those controlling protocol parameters or managing user funds. Consider using established solutions like OpenZeppelin's `TimelockController` for robustness and reliability. Alternatively, develop a custom timelock mechanism tailored to your specific requirements. Ensure that the timelock duration is appropriate for your contract's use case and stakeholder needs, providing sufficient time for review and intervention. Clearly document and communicate the presence and workings of the timelock mechanism to stakeholders to maintain transparency and trust in your contract's operations.

### Large transfers may not work with some `ERC20` tokens
Not all IERC20 implementations are totally compliant, and some (e.g UNI, COMP) may fail if the valued passed is larger than uint96. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)


```solidity
Path: ./src/managers/LockManager.sol

376:            token.transferFrom(_tokenOwner, address(this), _quantity);	// @audit-issue

423:            token.transfer(msg.sender, _quantity);	// @audit-issue
```
[376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L376-L376), [423](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L423-L423), 


#### Recommendation

When transferring tokens using ERC-20 contracts, such as UNI or COMP, it's important to be aware that not all IERC20 implementations are fully compliant, and they may not support large transfer values exceeding uint96. To avoid potential issues, it's essential to check the documentation or source code of the specific token you're using and ensure that your transfers adhere to the token's limitations.

### Possible loss of precision
Division by large numbers may result in precision loss due to rounding down, or even the result being erroneously equal to zero. Consider adding checks on the numerator to ensure precision loss is handled appropriately.


```solidity
Path: ./src/managers/LockManager.sol

364:                numberNFTs = (quantity - remainder) / configuredToken.nftCost;	// @audit-issue
```
[364](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L364-L364), 


#### Recommendation

Incorporate strategies in your Solidity contracts to mitigate precision loss in division operations. This can include:
1. Performing checks on the numerator and denominator to ensure they are within a reasonable range to avoid significant rounding errors.
2. Considering the use of fixed-point arithmetic libraries or scaling factors to handle divisions with higher precision.
3. Clearly documenting any inherent limitations of your division logic and providing guidelines for inputs to minimize unexpected behavior.
Always thoroughly test division operations under various scenarios to ensure that the outcomes are consistent with your contract's intended logic and accuracy requirements.

### Non-compliant `IERC20` tokens may revert with `transfer`
Some `IERC20` tokens (e.g. `BNB`, `OMG`, `USDT`) do not implement the standard properly, but they are still accepted by most code that accepts `ERC20` tokens.

For example, `USDT` transfer functions on L1 do not return booleans: when casted to `IERC20`, their function signatures do not match, and therefore the calls made will revert.

```solidity
Path: ./src/managers/LockManager.sol

376:            token.transferFrom(_tokenOwner, address(this), _quantity);	// @audit-issue

423:            token.transfer(msg.sender, _quantity);	// @audit-issue
```
[376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L376-L376), [423](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L423-L423), 


#### Recommendation

When dealing with tokens that may not fully comply with the `IERC20` standard, it's important to exercise caution and carefully verify the behavior of the specific token in question. In cases where tokens do not return booleans for transfer functions, consider implementing additional error-handling mechanisms to account for potential failures. This may include checking the token balance before and after the transfer to detect any discrepancies or using try-catch blocks to handle potential exceptions. Ensuring robust error handling can help your smart contract interact more gracefully with non-compliant tokens while maintaining security and reliability.

### Unsafe use of `transfer()`/`transferFrom()` with `IERC20`
Some tokens do not implement the `ERC20` standard properly but are still accepted by most code that accepts `ERC20` tokens. For example Tether (USDT)'s `transfer()` and `transferFrom()` functions on L1 do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to `IERC20`, their [function signatures](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca) do not match and therefore the calls made, revert (see [this](https://gist.github.com/IllIllI000/2b00a32e8f0559e8f386ea4f1800abc5) link for a test case). Use OpenZeppelin’s SafeERC20's `safeTransfer()`/`safeTransferFrom()` instead

```solidity
Path: ./src/managers/LockManager.sol

376:            token.transferFrom(_tokenOwner, address(this), _quantity);	// @audit-issue

423:            token.transfer(msg.sender, _quantity);	// @audit-issue
```
[376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L376-L376), [423](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L423-L423), 


#### Recommendation

When dealing with tokens that may not fully comply with the `IERC20` standard, it's important to exercise caution and carefully verify the behavior of the specific token in question. In cases where tokens do not return booleans for transfer functions, consider implementing additional error-handling mechanisms to account for potential failures. This may include checking the token balance before and after the transfer to detect any discrepancies or using try-catch blocks to handle potential exceptions. Ensuring robust error handling can help your smart contract interact more gracefully with non-compliant tokens while maintaining security and reliability.

### Unneeded initializations of integer variable to `0`.
In Solidity, it is common practice to initialize variables with default values when declaring them. However, initializing integer variables to `0` when they are not subsequently used in the code can lead to unnecessary gas consumption and code clutter. This issue points out instances where such initializations are present but serve no functional purpose.

```solidity
Path: ./src/managers/LockManager.sol

464:        uint256 lockedWeighted = 0;	// @audit-issue
```
[464](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L464-L464), 


#### Recommendation


It is recommended not to initialize integer variables to `0` to save some Gas.


### The `nonReentrant modifier` should occur before all other modifiers
In Solidity, the order in which modifiers are applied to a function can significantly impact the function's behavior and security. The `nonReentrant` modifier is crucial for preventing reentrancy attacks, a common vulnerability in smart contracts. This modifier should be the first in the list of applied modifiers to ensure that the reentrancy check is performed before any other logic in the function. Placing `nonReentrant` after other modifiers could potentially lead to security weaknesses, as other modifiers might perform state changes or external calls before the reentrancy protection takes effect. Therefore, correct positioning of the `nonReentrant` modifier is essential for maintaining the integrity and security of the function and, by extension, the entire contract.


```solidity
Path: ./src/managers/LockManager.sol

275:    function lockOnBehalf(
276:        address _tokenContract,
277:        uint256 _quantity,
278:        address _onBehalfOf
279:    )

297:    function lock(
298:        address _tokenContract,
299:        uint256 _quantity
300:    )

401:    function unlock(
402:        address _tokenContract,
403:        uint256 _quantity
404:    ) external notPaused nonReentrant {	// @audit-issue
```
[285](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L275-L279), [306](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L297-L300), [404](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L401-L404), 


#### Recommendation

To ensure the effectiveness of the `nonReentrant` modifier in protecting against reentrancy, it should be placed as the first modifier in a function's list of modifiers, before all other modifiers. This helps prevent reentrancy attacks from being triggered by other modifiers.

### Excessive Authorization in Contract Functions
A contract where a high percentage of functions require authorization (e.g., restricted to the contract owner or specific roles) may indicate over-centralization or excessive control. This could limit the contract's flexibility, reduce trust among users, and potentially create bottleneck points that could be exploited or become failure points. While some level of control is necessary for administrative purposes, overly restrictive access can detract from the decentralized nature of blockchain applications and concentrate too much power in the hands of a few.

```solidity
Path: ./src/managers/LockManager.sol

14:contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {	// @audit-issue: %81.81818181818183 amount of external/public and non-view/non-pure functions are required authorization to call.
	List of total functions: `disapproveUSDPrice`, `setUSDThresholds`, `lockOnBehalf`, `setLockDuration`, `configUpdated`, `lock`, `proposeUSDPrice`, `configureToken`, `unlock`, `configureLockdrop`, `approveUSDPrice`
	List of functions that require authorization: `disapproveUSDPrice`, `setUSDThresholds`, `lockOnBehalf`, `configUpdated`, `lock`, `proposeUSDPrice`, `configureToken`, `configureLockdrop`, `approveUSDPrice`
	List of functions that doesn't require authorization: `unlock`, `setLockDuration`
```
[14](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L14-L14), 


#### Recommendation

Make contract more decentralized.

### `if`-statement can be converted to a ternary
The code can be made more compact while also increasing readability by converting the following `if`-statements to ternaries (e.g. `foo += (x > y) ? a : b`)

```solidity
Path: ./src/managers/LockManager.sol

120:        if (configuredTokens[_tokenContract].nftCost == 0) {	// @audit-issue
121:            // new token
122:            configuredTokenContracts.push(_tokenContract);
123:        }

202:        if (usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD) {	// @audit-issue
203:            _execUSDPriceUpdate();
204:        }

289:        if (_onBehalfOf != address(0)) {	// @audit-issue
290:            lockRecipient = _onBehalfOf;
291:        }

349:        if (_lockDuration == 0) {	// @audit-issue
350:            _lockDuration = lockdrop.minLockDuration;
351:        }
```
[120](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L120-L123), [202](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L202-L204), [289](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L289-L291), [349](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L349-L351), 


#### Recommendation

Consider using single line if statements as ternary.

### Consider moving `msg.sender` checks to `modifier`s
If some functions are only allowed to be called by some specific users, consider using a modifier instead of checking with a require statement, especially if this check is done in multiple functions.

```solidity
Path: ./src/managers/LockManager.sol

192:        if (usdUpdateProposal.proposer == msg.sender)
193:            revert ProposerCannotApproveError();	// @audit-issue

194:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)
195:            revert ProposalAlreadyApprovedError();	// @audit-issue

225:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)
226:            revert ProposalAlreadyApprovedError();	// @audit-issue

227:        if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)
228:            revert ProposalAlreadyDisapprovedError();	// @audit-issue

256:                if (
257:                    uint32(block.timestamp) + uint32(_duration) <
258:                    lockedTokens[msg.sender][tokenContract].unlockTime
259:                ) {
260:                    revert LockDurationReducedError();	// @audit-issue
```
[193](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L192-L193), [195](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L194-L195), [226](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L225-L226), [228](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L227-L228), [260](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L256-L260), 


#### Recommendation

Consider refactoring your code by moving `msg.sender` checks to modifiers when certain functions are only allowed to be called by specific users. This approach can enhance code readability, reduce redundancy, and make it easier to maintain access control logic.

### Constants in comparisons should appear on the left side
Doing so will prevent [typo bugs](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html)

```solidity
Path: ./src/managers/LockManager.sol

48:        if (configuredTokens[_tokenContract].nftCost == 0)	// @audit-issue

119:        if (_tokenData.nftCost == 0) revert NFTCostInvalidError();	// @audit-issue

120:        if (configuredTokens[_tokenContract].nftCost == 0) {	// @audit-issue

159:        if (_contracts.length == 0) revert ProposalInvalidContractsError();	// @audit-issue

322:        if (_player.registrationDate == 0) revert AccountNotRegisteredError();	// @audit-issue

327:            if (msg.value != 0) revert InvalidMessageValueError();	// @audit-issue

349:        if (_lockDuration == 0) {	// @audit-issue

514:                if (configuredTokens[tokenContract].nftCost != 0) {	// @audit-issue
```
[48](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L48-L48), [119](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L119-L119), [120](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L120-L120), [159](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L159-L159), [322](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L322-L322), [327](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L327-L327), [349](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L349-L349), [514](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L514-L514), 


#### Recommendation

To prevent typo bugs and improve code readability, it's advisable to place constants on the left side of comparisons. This coding practice helps catch accidental assignment (=) instead of comparison (==) and enhances code quality.

### Style guide: Function ordering does not follow the Solidity style guide
According to the Solidity [style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

```solidity
Path: ./src/managers/LockManager.sol

85:    function configUpdated() external override onlyConfigStorage {	// @audit-issue
```
[85](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L85-L85), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/latest/style-guide.html).

### Duplicated `require()`/`revert()` checks should be refactored to a modifier or function
In Solidity contracts, it's common to encounter duplicated `require()` or `revert()` statements across multiple functions. These statements are crucial for validating conditions and ensuring contract integrity. However, repeated checks can lead to code redundancy, making the contract larger and more difficult to maintain. This redundancy can be streamlined by encapsulating the repeated logic in a modifier or a dedicated function. By doing so, the code becomes more concise, easier to audit, and more gas-efficient. Furthermore, centralizing the validation logic in a single location makes the codebase more adaptable to changes and reduces the risk of inconsistencies or errors in future updates.

```solidity
Path: ./src/managers/LockManager.sol

133:        if (usdUpdateProposal.proposer != address(0))	// @audit-issue: Same if statement in line(s): ['157']

191:        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();	// @audit-issue: Same if statement in line(s): ['224']

194:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)	// @audit-issue: Same if statement in line(s): ['225']

196:        if (usdUpdateProposal.proposedPrice != _price)	// @audit-issue: Same if statement in line(s): ['229']
```
[133](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L133-L133), [191](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L191-L191), [194](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L194-L194), [196](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L196-L196), 


#### Recommendation

Consolidate repeated `require()` or `revert()` checks in Solidity by creating a modifier or a separate function. Apply this modifier to all functions requiring the same validation, or call the function at the beginning of each relevant function. This refactoring enhances contract efficiency, maintainability, and consistency, while potentially reducing gas costs associated with deploying and executing redundant code.

### Duplicate import statements
Contracts often depend on libraries and other contracts to modularize code and reuse functionalities. However, redundant imports occur when a contract imports a library or another contract that is already imported by one of its dependencies. This redundancy does not impact the compiled bytecode but can clutter the codebase, making it harder to understand the direct dependencies of each contract. Simplifying imports by removing these redundancies enhances code readability and maintainability.

```solidity
Path: ./src/managers/LockManager.sol

7:import "../interfaces/IConfigStorage.sol";	// @audit-issue: Same library is also imported on: `['BaseBlastManager']`, at lines: `[5]`
```
[7](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L7-L7), 


#### Recommendation

Review your Solidity contracts and eliminate duplicate import statements. Ensure each file is imported only once where it's needed. If you find the same import statement across multiple files, consider whether you can restructure your code to centralize common logic or dependencies, potentially through base contracts or libraries. Regularly conduct code audits or utilize linters and other static analysis tools to identify and resolve duplicate imports, thereby enhancing the clarity, structure, and security of your Solidity codebase.

### Array is `push()`ed but not `pop()`ed
Array entries are added but are never removed. Consider whether this should be the case, or whether there should be a maximum, or whether old entries should be removed. Cases where there are specific potential problems will be flagged separately under a different issue.

```solidity
Path: ./src/managers/LockManager.sol

122:            configuredTokenContracts.push(_tokenContract);	// @audit-issue
```
[122](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L122-L122), 


#### Recommendation

When working with arrays in your Solidity code, carefully evaluate whether the addition of array entries using `push()` should be balanced with corresponding removal using methods like `pop()`, `splice()`, or other appropriate array manipulation techniques. This evaluation is crucial to manage the array's size and prevent unnecessary storage growth, which can impact gas costs and contract efficiency.

### Too long functions should be refactored
Functions with too many lines are difficult to understand. It is recommended to refactor complex functions into multiple shorter and easier to understand functions.


```solidity
Path: ./src/managers/LockManager.sol

311:    function _lock(	// @audit-issue 87 lines
```
[311](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L311-L311), 


#### Recommendation

To address this issue, refactor long and complex functions into multiple shorter and more manageable functions. This will improve code readability and maintainability, making it easier to understand and maintain your smart contract.

### Dependence on external protocols
External protocols should be monitored as such dependencies may introduce vulnerabilities if a vulnerability is found /introduced in the external protocol

```solidity
Path: ./src/managers/LockManager.sol

4:import "@openzeppelin/contracts/token/ERC20/ERC20.sol";	// @audit-issue

5:import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";	// @audit-issue
```
[4](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L4-L4), [5](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L5-L5), 


#### Recommendation

Regularly monitor and review the external protocols your Solidity contracts depend on for any updates or identified vulnerabilities. Consider implementing fallback mechanisms or contingency plans in your contracts to handle potential failures or compromises in these external protocols. Additionally, thoroughly audit and test the integration points with external protocols to ensure they adhere to your contract's security standards. Where possible, design your contract architecture to minimize reliance on external protocols or allow for upgradability in response to changes in these dependencies. Stay informed about developments in the protocols you depend on and actively participate in their community for early awareness of potential issues.

### UPPER_CASE names should be reserved for `constant`/`immutable` variables
In Solidity, and in programming in general, naming conventions are essential for readability and understanding of the code. It's a common practice to use UPPER_CASE naming for `constant` or `immutable` variables. These are variables whose values are set at contract deployment and do not change thereafter. Adhering to this convention helps distinguish `constant` or `immutable` variables from other variables whose values may change. This practice enhances code clarity and makes it easier to understand the contract's behavior, especially for new developers or auditors reviewing the code. Using UPPER_CASE for non-constant variables can mislead and cause confusion about the nature of these variables.

```solidity
Path: ./src/managers/LockManager.sol

16:    uint8 APPROVE_THRESHOLD = 3;	// @audit-issue

18:    uint8 DISAPPROVE_THRESHOLD = 3;	// @audit-issue
```
[16](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L16-L16), [18](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L18-L18), 


#### Recommendation

Adopt a clear and consistent naming convention in your Solidity contracts by using UPPER_CASE names exclusively for `constant` or `immutable` variables. Review your codebase to ensure that this convention is consistently applied, and refactor any non-constant variables named in UPPER_CASE to a different naming style. This will enhance the readability and clarity of your code, making it more maintainable and easier to understand at a glance.

### Style Guide: Surround top level declarations in Solidity source with two blank lines.
1- Surround top level declarations in Solidity source with two blank lines.
2- Within a contract surround function declarations with a single blank line.


```solidity
Path: ./src/managers/LockManager.sol

44:    INFTOverlord public nftOverlord;
45:
46:    /// @notice Token supplied must be configured but can be inactive	// @audit-issue: There should be single blank line between function declarations.
47:    modifier onlyConfiguredToken(address _tokenContract) {

51:    }
52:
53:    /// @notice Token supplied must be configured and active	// @audit-issue: There should be single blank line between function declarations.
54:    modifier onlyActiveToken(address _tokenContract) {

95:    }
96:
97:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
98:    function configureLockdrop(

112:    }
113:
114:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
115:    function configureToken(

139:    }
140:
141:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
142:    function proposeUSDPrice(

174:    }
175:
176:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
177:    function approveUSDPrice(

207:    }
208:
209:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
210:    function disapproveUSDPrice(

242:    }
243:
244:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
245:    function setLockDuration(uint256 _duration) external notPaused {

272:    }
273:
274:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
275:    function lockOnBehalf(

294:    }
295:
296:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
297:    function lock(

398:    }
399:
400:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
401:    function unlock(

427:    }
428:
429:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
430:    function getLocked(

458:    }
459:
460:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
461:    function getLockedWeightedValue(

487:    }
488:
489:    /// @inheritdoc ILockManager	// @audit-issue: There should be single blank line between function declarations.
490:    function getConfiguredToken(

500:    }
501:
502:    /*******************************************************
503:     ** INTERNAL FUNCTIONS
504:     ********************************************************/
505:	// @audit-issue: There should be single blank line between function declarations.
506:    function _execUSDPriceUpdate() internal {
```
[46](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L44-L47), [53](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L51-L54), [97](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L95-L98), [114](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L112-L115), [141](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L139-L142), [176](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L174-L177), [209](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L207-L210), [244](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L242-L245), [274](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L272-L275), [296](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L294-L297), [400](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L398-L401), [429](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L427-L430), [460](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L458-L461), [489](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L487-L490), [505](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L500-L506), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/latest/style-guide.html#blank-lines).

### `constants` should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

```solidity
Path: ./src/managers/LockManager.sol

482:                    1e18;	// @audit-issue
```
[482](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L482-L482), 


#### Recommendation

Consider defining constants with meaningful names for magic numbers and hexadecimal literals to improve code readability and maintainability.

### Lack of index element validation in function
There's no validation to check whether the index element provided as an argument actually exists in the call. This omission could lead to unintended behavior if an element that does not exist in the call is passed to the function. The function should validate that the provided index element exists in the call before proceeding.

```solidity
Path: ./src/managers/LockManager.sol

253:            address tokenContract = configuredTokenContracts[i];	// @audit-issue

454:                configuredTokenContracts[i]	// @audit-issue

480:                        configuredTokens[configuredTokenContracts[i]]	// @audit-issue
```
[253](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L253-L253), [454](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L454-L454), [480](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L480-L480), 


#### Recommendation

Integrate explicit index validation checks at the beginning of functions that operate based on index elements. Use conditional statements to verify that the provided index falls within the valid range of existing elements. For array operations, ensure the index is less than the array's length. For mappings, consider additional logic to confirm the presence of a key. For example, in an array-based function:
```solidity
function getElementByIndex(uint256 index) public view returns (ElementType) {
    require(index < array.length, "Index out of bounds");
    return array[index];
}
```


### Contract timekeeping will break earlier than the Ethereum network itself will stop working
When a timestamp is downcast from `uint256` to `uint32`, the value will wrap in the year 2106, and the contracts will break. Other downcasts will have different endpoints. Consider whether your contract is intended to live past the size of the type being used.

```solidity
Path: ./src/managers/LockManager.sol

104:                uint32(block.timestamp)	// @audit-issue

166:        usdUpdateProposal.proposedDate = uint32(block.timestamp);	// @audit-issue

257:                    uint32(block.timestamp) + uint32(_duration) <	// @audit-issue

353:            lockdrop.start <= uint32(block.timestamp) &&	// @audit-issue

354:            lockdrop.end >= uint32(block.timestamp)	// @audit-issue

381:        lockedToken.lastLockTime = uint32(block.timestamp);	// @audit-issue

383:            uint32(block.timestamp) +	// @audit-issue

410:        if (lockedToken.unlockTime > uint32(block.timestamp))	// @audit-issue
```
[104](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L104-L104), [166](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L166-L166), [257](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L257-L257), [353](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L353-L353), [354](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L354-L354), [381](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L381-L381), [383](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L383-L383), [410](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L410-L410), 


#### Recommendation

Carefully consider the choice of data types for timestamps in your Solidity contracts. If downcasting from `uint256` to smaller types like `uint32`, be aware of the overflow risks and the potential for the contract to break prematurely. For contracts intended to operate beyond these time limits, use larger integer types capable of representing dates far into the future. Additionally, document any decisions around the use of specific timestamp types and their implications for the contract's longevity and reliability. Regularly review and update your contracts to ensure that they remain robust against future changes in the Ethereum network and the broader blockchain ecosystem.

### Variable names that consist of all capital letters should be reserved for constant/immutable variables
If the variable needs to be different based on which class it comes from, a `view`/`pure` function should be used instead (e.g. like [this](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59)).


```solidity
Path: ./src/managers/LockManager.sol

16:    uint8 APPROVE_THRESHOLD = 3;	// @audit-issue

18:    uint8 DISAPPROVE_THRESHOLD = 3;	// @audit-issue
```
[16](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L16-L16), [18](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L18-L18), 


#### Recommendation

Variable names consisting of all capital letters should be reserved for constant or immutable variables. If the variable's value needs to vary based on the class it comes from, consider using a `view` or `pure` function instead of a constant variable. This allows for dynamic computation of the value when needed, ensuring that it reflects the appropriate state or context.

### Inefficient Array Usage
Use mappings instead of arrays for managing lists of data in order to conserve gas. Mappings are less expensive and more efficient for accessing any value without having to iterate through an array.

```solidity
Path: ./src/managers/LockManager.sol

22:    address[] public configuredTokenContracts;	// @audit-issue

144:        address[] calldata _contracts	// @audit-issue

434:        LockedTokenWithMetadata[]	// @audit-issue
```
[22](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L22-L22), [144](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L144-L144), [434](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L434-L434), 


#### Recommendation

In scenarios where data access efficiency is critical, prefer using mappings over arrays in Solidity contracts. Mappings offer more efficient and gas-effective data retrieval and updates, especially when dealing with large or frequently accessed datasets. Ensure to structure your data and choose keys thoughtfully to maximize the efficiency gains offered by mappings. While arrays might be suitable for ordered data or when the entire dataset needs to be iterated, for most other use cases, mappings are likely to be the more gas-efficient choice.

### Lack of specific import identifier
It is better to use `import {<identifier>} from "<file.sol>"` instead of `import "<file.sol>"` to improve readability and speed up the compilation time.

```solidity
Path: ./src/managers/LockManager.sol

4:import "@openzeppelin/contracts/token/ERC20/ERC20.sol";	// @audit-issue

5:import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";	// @audit-issue

6:import "../interfaces/ILockManager.sol";	// @audit-issue

7:import "../interfaces/IConfigStorage.sol";	// @audit-issue

8:import "../interfaces/IAccountManager.sol";	// @audit-issue

9:import "../interfaces/IMigrationManager.sol";	// @audit-issue

10:import "./BaseBlastManager.sol";	// @audit-issue

11:import "../interfaces/ISnuggeryManager.sol";	// @audit-issue

12:import "../interfaces/INFTOverlord.sol";	// @audit-issue
```
[4](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L4-L4), [5](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L5-L5), [6](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L6-L6), [7](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L7-L7), [8](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L8-L8), [9](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L9-L9), [10](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L10-L10), [11](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L11-L11), [12](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L12-L12), 


#### Recommendation

To improve code clarity and avoid naming conflicts, it's recommended to use specific import identifiers when importing from other contracts or libraries. Instead of using `import "<file.sol>";`, specify the desired identifiers using `import { <identifier1>, <identifier2> } from "<file.sol>";`. This not only enhances readability but also can speed up compilation times by only importing the necessary symbols.

### Some variables have a implicit default visibility
Consider always adding an explicit visibility modifier for variables, as the default is `internal`.

```solidity
Path: ./src/managers/LockManager.sol

16:    uint8 APPROVE_THRESHOLD = 3;	// @audit-issue

18:    uint8 DISAPPROVE_THRESHOLD = 3;	// @audit-issue

26:    mapping(address => PlayerSettings) playerSettings;	// @audit-issue

30:    USDUpdateProposal usdUpdateProposal;	// @audit-issue
```
[16](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L16-L16), [18](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L18-L18), [26](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L26-L26), [30](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L30-L30), 


#### Recommendation

Always add an explicit visibility modifier for variables to enhance code clarity and avoid potential issues. The default visibility for variables is `internal`, but specifying it explicitly makes your intentions clear.

### Ensure Events Emission Prior to External Calls to Prevent Out-of-Order Issues
It's essential to ensure that events follow the best practice of check-effects-interaction and are emitted before any external calls to prevent out-of-order events due to reentrancy. Emitting events post external interactions may cause them to be out of order due to reentrancy, which can be misleading or erroneous for event listeners. [Refer to the Solidity Documentation for best practices](https://solidity.readthedocs.io/en/latest/security-considerations.html#reentrancy).

```solidity
Path: ./src/managers/LockManager.sol

420:            payable(msg.sender).transfer(_quantity);
421:        } else {
422:            IERC20 token = IERC20(_tokenContract);
423:            token.transfer(msg.sender, _quantity);
424:        }
425:
426:        emit Unlocked(msg.sender, _tokenContract, _quantity);	// @audit-issue
```
[426](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L420-L426), 


#### Recommendation

To prevent out-of-order events and ensure consistency, always emit events before making any external calls or interactions within your smart contracts. This practice adheres to the check-effects-interaction pattern and helps provide a clear and accurate event log for event listeners. Following this approach enhances the reliability and predictability of your smart contract's behavior.

### Unnecessary Use of override Keyword
In Solidity version 0.8.8 and later, the use of the override keyword becomes superfluous when a function is overriding solely from an interface and the function isn't present in multiple base contracts. Previously, the override keyword was required as an explicit indication to the compiler. However, this is no longer the case, and the extraneous use of the keyword can make the code less clean and more verbose.
Solidity documentation on [Function Overriding](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding).


```solidity
Path: ./src/managers/LockManager.sol

85:    function configUpdated() external override onlyConfigStorage {	// @audit-issue
```
[85](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L85-L85), 


#### Recommendation

In Solidity versions 0.8.8 and later, the `override` keyword is no longer required for functions that are solely overriding from an interface and not present in multiple base contracts. Removing the unnecessary `override` keyword can make the code cleaner and less verbose.

### Include sender information in events
When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the `msg.sender` the events of these types of action will make events much more useful to end users, especially when `msg.sender` is not `tx.origin`.

```solidity
Path: ./src/managers/LockManager.sol

111:        emit LockDropConfigured(_lockdropData);	// @audit-issue

126:        emit TokenConfigured(_tokenContract, _tokenData);	// @audit-issue

138:        emit USDThresholdUpdated(_approve, _disapprove);	// @audit-issue

240:            emit RemovedUSDProposal();	// @audit-issue

389:        emit Locked(	// @audit-issue

518:                    emit USDPriceUpdated(	// @audit-issue
```
[111](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L111-L111), [126](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L126-L126), [138](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L138-L138), [240](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L240-L240), [389](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L389-L389), [518](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L518-L518), 


#### Recommendation

To improve the usability and analysis of your smart contract events, consider including the `msg.sender` address as part of the event data. This enables easier filtering and identification of the sender's actions within your contract, providing valuable insights for users and external tools.

### Avoid external calls in modifiers
It is unusual to have external calls in modifiers, and doing so will make reviewers more likely to miss important external interactions. Consider moving the external call to an internal function, and calling that function from the modifier.

```solidity
Path: ./src/managers/LockManager.sol

48:        if (configuredTokens[_tokenContract].nftCost == 0)	// @audit-issue

55:        if (!configuredTokens[_tokenContract].active)	// @audit-issue
```
[48](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L48-L48), [55](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L55-L55), 


#### Recommendation

Refrain from incorporating external calls directly within modifiers. Instead, encapsulate the external call within an internal function and invoke this function from within the body of the functions that use the modifier. This approach enhances code readability and security, making it easier for reviewers and auditors to track external interactions. Additionally, it centralizes external calls, simplifying the management and review of these potentially risky operations. Always ensure external calls are handled with care, implementing checks, balances, and reentrancy guards as necessary to protect your contract from malicious actors and unintended consequences.

### Control structures do not follow the Solidity Style Guide
Refer to the [Solidity style guide - Control Structures](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#control-structures).

```solidity
Path: ./src/managers/LockManager.sol

48:        if (configuredTokens[_tokenContract].nftCost == 0)	// @audit-issue

55:        if (!configuredTokens[_tokenContract].active)	// @audit-issue

101:        if (_lockdropData.end < block.timestamp)	// @audit-issue

106:        if (_lockdropData.start >= _lockdropData.end)	// @audit-issue

133:        if (usdUpdateProposal.proposer != address(0))	// @audit-issue

142:    function proposeUSDPrice(
143:        uint256 _price,
144:        address[] calldata _contracts
145:    )	// @audit-issue

157:        if (usdUpdateProposal.proposer != address(0))	// @audit-issue

177:    function approveUSDPrice(
178:        uint256 _price
179:    )	// @audit-issue

192:        if (usdUpdateProposal.proposer == msg.sender)	// @audit-issue

194:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)	// @audit-issue

196:        if (usdUpdateProposal.proposedPrice != _price)	// @audit-issue

210:    function disapproveUSDPrice(
211:        uint256 _price
212:    )	// @audit-issue

225:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)	// @audit-issue

227:        if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)	// @audit-issue

229:        if (usdUpdateProposal.proposedPrice != _price)	// @audit-issue

246:        if (_duration > configStorage.getUint(StorageKey.MaxLockDuration))	// @audit-issue

256:                if (
257:                    uint32(block.timestamp) + uint32(_duration) <
258:                    lockedTokens[msg.sender][tokenContract].unlockTime	// @audit-issue

275:    function lockOnBehalf(
276:        address _tokenContract,
277:        uint256 _quantity,
278:        address _onBehalfOf
279:    )	// @audit-issue

297:    function lock(
298:        address _tokenContract,
299:        uint256 _quantity
300:    )	// @audit-issue

352:        if (
353:            lockdrop.start <= uint32(block.timestamp) &&
354:            lockdrop.end >= uint32(block.timestamp)	// @audit-issue

356:            if (
357:                _lockDuration < lockdrop.minLockDuration ||
358:                _lockDuration >
359:                uint32(configStorage.getUint(StorageKey.MaxLockDuration))	// @audit-issue

408:        if (lockedToken.quantity < _quantity)	// @audit-issue

410:        if (lockedToken.unlockTime > uint32(block.timestamp))	// @audit-issue

467:            if (
468:                lockedTokens[_player][configuredTokenContracts[i]].quantity >
469:                0 &&
470:                configuredTokens[configuredTokenContracts[i]].active	// @audit-issue

507:        if (
508:            usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD &&
509:            usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD	// @audit-issue
```
[48](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L48-L48), [55](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L55-L55), [101](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L101-L101), [106](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L106-L106), [133](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L133-L133), [145](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L142-L145), [157](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L157-L157), [179](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L177-L179), [192](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L192-L192), [194](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L194-L194), [196](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L196-L196), [212](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L210-L212), [225](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L225-L225), [227](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L227-L227), [229](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L229-L229), [246](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L246-L246), [258](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L256-L258), [279](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L275-L279), [300](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L297-L300), [354](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L352-L354), [359](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L356-L359), [408](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L408-L408), [410](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L410-L410), [470](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L467-L470), [509](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L507-L509), 


#### Recommendation

Adhere to the Solidity style guide regarding control structures by avoiding the definition of multiple functions with identical names in a contract. Unique and descriptive function names improve code clarity and prevent potential confusion or errors. Consult [Solidity style guide - Control Structures](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#control-structures) for best practices.

### Consider making contracts `Upgradeable`
This allows for bugs to be fixed in production, at the expense of significantly increasing centralization.

```solidity
Path: ./src/managers/LockManager.sol

14:contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {	// @audit-issue
```
[14](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L14-L14), 


#### Recommendation

Assess the need for upgradeability in your Solidity contracts based on the project's requirements and lifecycle. If chosen, implement a well-known proxy pattern ensuring rigorous security and governance mechanisms are in place. Be aware of the increased centralization and plan accordingly to mitigate potential risks, such as through decentralized governance models or multi-sig control for upgrade decisions.

### Events that mark critical parameter changes should contain both the old and the new value
This should especially be done if the new value is not required to be different from the old value

```solidity
Path: ./src/managers/LockManager.sol

111:        emit LockDropConfigured(_lockdropData);	// @audit-issue

126:        emit TokenConfigured(_tokenContract, _tokenData);	// @audit-issue
```
[111](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L111-L111), [126](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L126-L126), 


#### Recommendation

To enhance transparency and auditability, ensure that events are emitted when sensitive changes are made to the contracts. Review and update functions that lack event emissions, especially in cases where sensitive operations or state changes occur, to provide a clear record of such actions.

### Non-`external`/`public` variable names should begin with an underscore
According to the Solidity Style Guide, non-external/public variable names should begin with an underscore


```solidity
Path: ./src/managers/LockManager.sol

16:    uint8 APPROVE_THRESHOLD = 3;	// @audit-issue

18:    uint8 DISAPPROVE_THRESHOLD = 3;	// @audit-issue

26:    mapping(address => PlayerSettings) playerSettings;	// @audit-issue

30:    USDUpdateProposal usdUpdateProposal;	// @audit-issue
```
[16](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L16-L16), [18](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L18-L18), [26](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L26-L26), [30](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L30-L30), 


#### Recommendation

To adhere to the Solidity Style Guide, consider prefixing the names of non-`external`/`public` variables with an underscore (_). This naming convention enhances code readability and helps distinguish the visibility of  variables.

### Revert statements within external and public functions can be used to perform DOS attacks
In Solidity, `revert` statements are used to undo changes and throw an exception when certain conditions are not met. However, in public and external functions, improper use of `revert` can be exploited for Denial of Service (DoS) attacks. An attacker can intentionally trigger these `revert' conditions, causing legitimate transactions to consistently fail. For example, if a function relies on specific conditions from user input or contract state, an attacker could manipulate these to continually force `revert`s, blocking the function's execution. Therefore, it's crucial to design contract logic to handle exceptions properly and avoid scenarios where `revert` can be predictably triggered by malicious actors. This includes careful input validation and considering alternative design patterns that are less susceptible to such abuses.

```solidity
Path: ./src/managers/LockManager.sol

247:            revert MaximumLockDurationError();	// @audit-issue

260:                    revert LockDurationReducedError();	// @audit-issue

409:            revert InsufficientLockAmountError();	// @audit-issue

411:            revert TokenStillLockedError();	// @audit-issue
```
[247](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L247-L247), [260](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L260-L260), [409](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L409-L409), [411](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L411-L411), 


#### Recommendation

Design your Solidity contract's public and external functions with care to mitigate the risk of DoS attacks via `revert` statements. Implement robust input validation to ensure inputs are within expected bounds and conditions. Consider alternative logic or design patterns that reduce the reliance on `revert` for critical operations, particularly those that can be influenced externally. Evaluate the use of modifiers, try-catch blocks, or state checks that allow for safer handling of conditions and exceptions. Ensure that your contract's critical functionality remains accessible and resilient against potential abuse of `revert` behavior by malicious actors.

### Consider adding emergency-stop functionality
In the event of a security breach or any unforeseen emergency, swiftly suspending all protocol operations becomes crucial. Having a mechanism in place to halt all functions collectively, instead of pausing individual contracts separately, substantially enhances the efficiency of mitigating ongoing attacks or vulnerabilities. This not only quickens the response time to potential threats but also reduces operational stress during these critical periods. Therefore, consider integrating a 'circuit breaker' or 'emergency stop' function into the smart contract system architecture. Such a feature would provide the capability to suspend the entire protocol instantly, which could prove invaluable during a time-sensitive crisis management situation.

```solidity
Path: ./src/managers/LockManager.sol

14:contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {	// @audit-issue
```
[14](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L14-L14), 


#### Recommendation

Implement an emergency-stop feature in your Solidity contract system to enhance security and crisis response capabilities. This can be achieved through a 'circuit breaker' pattern, where a central switch or set of conditions can instantly suspend critical operations across the contract ecosystem. Ensure that this mechanism is accessible to authorized parties only, such as contract administrators or a decentralized governance system. Design the emergency-stop functionality to be transparent and auditable, with clear conditions and processes for activation and deactivation. Regularly test and audit this feature to ensure its reliability and effectiveness in potential emergency situations.

### Cyclomatic complexity in functions
Cyclomatic complexity is a software metric used to measure the complexity of a program. It quantifies the number of linearly independent paths through a program's source code, giving an idea of how complex the control flow is. High cyclomatic complexity may indicate a higher risk of defects and can make the code harder to understand, test, and maintain. It often suggests that a function or method is trying to do too much, and a refactor might be needed. By breaking down complex functions into smaller, more focused pieces, you can improve readability, ease of testing, and overall maintainability.

```solidity
Path: ./src/managers/LockManager.sol

311:    function _lock(	// @audit-issue
312:        address _tokenContract,
313:        uint256 _quantity,
314:        address _tokenOwner,
315:        address _lockRecipient
316:    ) private {
317:        (
318:            address _mainAccount,
319:            MunchablesCommonLib.Player memory _player
320:        ) = accountManager.getPlayer(_lockRecipient);
321:        if (_mainAccount != _lockRecipient) revert SubAccountCannotLockError();
322:        if (_player.registrationDate == 0) revert AccountNotRegisteredError();
323:        // check approvals and value of tx matches
324:        if (_tokenContract == address(0)) {
325:            if (msg.value != _quantity) revert ETHValueIncorrectError();
326:        } else {
327:            if (msg.value != 0) revert InvalidMessageValueError();
328:            IERC20 token = IERC20(_tokenContract);
329:            uint256 allowance = token.allowance(_tokenOwner, address(this));
330:            if (allowance < _quantity) revert InsufficientAllowanceError();
331:        }
332:
333:        LockedToken storage lockedToken = lockedTokens[_lockRecipient][
334:            _tokenContract
335:        ];
336:        ConfiguredToken storage configuredToken = configuredTokens[
337:            _tokenContract
338:        ];
339:
340:        // they will receive schnibbles at the new rate since last harvest if not for force harvest
341:        accountManager.forceHarvest(_lockRecipient);
342:
343:        // add remainder from any previous lock
344:        uint256 quantity = _quantity + lockedToken.remainder;
345:        uint256 remainder;
346:        uint256 numberNFTs;
347:        uint32 _lockDuration = playerSettings[_lockRecipient].lockDuration;
348:
349:        if (_lockDuration == 0) {
350:            _lockDuration = lockdrop.minLockDuration;
351:        }
352:        if (
353:            lockdrop.start <= uint32(block.timestamp) &&
354:            lockdrop.end >= uint32(block.timestamp)
355:        ) {
356:            if (
357:                _lockDuration < lockdrop.minLockDuration ||
358:                _lockDuration >
359:                uint32(configStorage.getUint(StorageKey.MaxLockDuration))
360:            ) revert InvalidLockDurationError();
361:            if (msg.sender != address(migrationManager)) {
362:                // calculate number of nfts
363:                remainder = quantity % configuredToken.nftCost;
364:                numberNFTs = (quantity - remainder) / configuredToken.nftCost;
365:
366:                if (numberNFTs > type(uint16).max) revert TooManyNFTsError();
367:
368:                // Tell nftOverlord that the player has new unopened Munchables
369:                nftOverlord.addReveal(_lockRecipient, uint16(numberNFTs));
370:            }
371:        }
372:
373:        // Transfer erc tokens
374:        if (_tokenContract != address(0)) {
375:            IERC20 token = IERC20(_tokenContract);
376:            token.transferFrom(_tokenOwner, address(this), _quantity);
377:        }
378:
379:        lockedToken.remainder = remainder;
380:        lockedToken.quantity += _quantity;
381:        lockedToken.lastLockTime = uint32(block.timestamp);
382:        lockedToken.unlockTime =
383:            uint32(block.timestamp) +
384:            uint32(_lockDuration);
385:
386:        // set their lock duration in playerSettings
387:        playerSettings[_lockRecipient].lockDuration = _lockDuration;
388:
389:        emit Locked(
390:            _lockRecipient,
391:            _tokenOwner,
392:            _tokenContract,
393:            _quantity,
394:            remainder,
395:            numberNFTs,
396:            _lockDuration
397:        );
398:    }
```
[311](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L311-L398), 


#### Recommendation

Regularly analyze your Solidity contracts for functions with high cyclomatic complexity. Consider refactoring these functions into smaller, more focused units. This could involve breaking down complex functions into multiple simpler functions, reducing the number of conditional branches, or simplifying logic where possible. Such refactoring improves the readability and testability of your code and reduces the likelihood of defects. Additionally, simpler functions are easier to audit and maintain, enhancing the overall security and robustness of your contract. Employ tools or metrics to periodically assess the complexity of your functions as part of your development and maintenance processes.

### Consider only defining one library/interface/contract per sol file
Combining multiple libraries, interfaces, or contracts in a single .sol file can lead to clutter, reduced readability, and versioning issues. Resolution: Adopt the best practice of defining only one library, interface, or contract per Solidity file. This modular approach enhances clarity, simplifies unit testing, and streamlines code review. Furthermore, segregating components makes version management easier, as updates to one component won't necessitate changes to a file housing multiple unrelated components. Structured file management can further assist in avoiding naming collisions and ensure smoother integration into larger systems or DApps.

```solidity
Path: ./src/managers/LockManager.sol

4:import "@openzeppelin/contracts/token/ERC20/ERC20.sol";	// @audit-issue
5:import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";
6:import "../interfaces/ILockManager.sol";
7:import "../interfaces/IConfigStorage.sol";
8:import "../interfaces/IAccountManager.sol";
9:import "../interfaces/IMigrationManager.sol";
10:import "./BaseBlastManager.sol";
11:import "../interfaces/ISnuggeryManager.sol";
12:import "../interfaces/INFTOverlord.sol";
```
[4](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L4-L12), 


#### Recommendation

Adopt a modular file structure in your Solidity projects by defining only one library, interface, or contract per file. This approach significantly enhances the clarity and readability of your code, making it easier to manage, test, and review. It also simplifies version control, as updates to individual components are isolated to their respective files, reducing the risk of unintended side effects. Organize your project directory to reflect this structure, with a clear naming convention that matches file names to their contained contracts, libraries, or interfaces. This organization not only prevents naming collisions but also facilitates smoother integration into larger systems or decentralized applications (DApps).

### Reduce deployment costs by tweaking contracts' metadata
When solidity generates the bytecode for the smart contract to be deployed, it appends metadata about the compilation at the end of the bytecode.
By default, the solidity compiler appends metadata at the end of the “actual” initcode, which gets stored to the blockchain when the constructor finishes executing.
Consider tweaking the metadata to avoid this unnecessary allocation. A full guide can be found [here](https://www.rareskills.io/post/solidity-metadata).
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L1), 


#### Recommendation

See [this](https://www.rareskills.io/post/solidity-metadata) link, at its bottom, for full details

### All verbatim blocks are considered identical by deduplicator and can incorrectly be unified
The Solidity Team reported a bug on October 24, 2023, affecting Yul code using the verbatim builtin, specifically in the Block Deduplicator optimizer step. This bug, present since Solidity version 0.8.5, caused incorrect deduplication of verbatim assembly items surrounded by identical opcodes, considering them identical regardless of their data. The bug was confined to pure Yul compilation with optimization enabled and was unlikely to be exploited as an attack vector. The conditions triggering the bug were very specific, and its occurrence was deemed to have a low likelihood. The bug was rated with an overall low score due to these factors.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L1), 


#### Recommendation

Review and assess any Solidity contracts, especially those involving Yul code, that may be impacted by this deduplication bug. If your contracts rely on the Block Deduplicator optimizer and use verbatim blocks in a way that could be affected by this issue, consider updating your Solidity version to one where this bug is fixed, or adjust your contract to avoid this specific scenario. Stay informed about updates from the Solidity Team regarding this and similar issues, and regularly update your Solidity compiler to the latest version to benefit from bug fixes and optimizations. Given the specific and limited nature of this bug, its impact may be minimal, but caution is advised for contracts with complex assembly code or those heavily reliant on optimizer behaviors.

### Consider adding formal verification proofs
Formal verification is the act of proving or disproving the correctness of intended algorithms underlying a system with respect to a certain formal specification/property/invariant, using formal methods of mathematics.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L1), 


#### Recommendation

Consider integrating formal verification into your Solidity contract development process. This can be done by defining formal specifications and properties that your contract should adhere to and using mathematical methods to verify these aspects. Tools and platforms like Certora Prover, Scribble, or OpenZeppelin's test environment can assist in this process. Formal verification should complement traditional testing and auditing methods, offering an additional layer of security assurance. Keep in mind that formal verification requires a thorough understanding of mathematical logic and contract specifications, so it may necessitate additional resources or expertise. Nevertheless, the investment in formal verification can significantly enhance the trustworthiness and robustness of your smart contracts.

### Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L1), 


#### Recommendation

Consider writing test cases.

### Large or complicated code bases should implement invariant tests
This includes: large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts. Invariant fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and invariant fuzzers may help significantly.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L1), 


#### Recommendation

Consider writing invariant test cases.

### Consider adding formal verification proofs
Consider using formal verification to mathematically prove that your code does what is intended, and does not have any edge cases with unexpected behavior. The solidity compiler itself has this functionality [built in](https://docs.soliditylang.org/en/latest/smtchecker.html#smtchecker-and-formal-verification)
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L1), 


#### Recommendation

Consider using formal verification.

### NatSpec: Body of `if` statement should be placed on a new line
According to the [Solidity style guide](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures), `if` statements whose body contains a single line should look like this: solidity `if (x < 10)     x += 1; `

```solidity
Path: ./src/managers/LockManager.sol

48:        if (configuredTokens[_tokenContract].nftCost == 0)	// @audit-issue

55:        if (!configuredTokens[_tokenContract].active)	// @audit-issue

101:        if (_lockdropData.end < block.timestamp)	// @audit-issue

106:        if (_lockdropData.start >= _lockdropData.end)	// @audit-issue

119:        if (_tokenData.nftCost == 0) revert NFTCostInvalidError();	// @audit-issue

133:        if (usdUpdateProposal.proposer != address(0))	// @audit-issue

157:        if (usdUpdateProposal.proposer != address(0))	// @audit-issue

159:        if (_contracts.length == 0) revert ProposalInvalidContractsError();	// @audit-issue

191:        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();	// @audit-issue

192:        if (usdUpdateProposal.proposer == msg.sender)	// @audit-issue

194:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)	// @audit-issue

196:        if (usdUpdateProposal.proposedPrice != _price)	// @audit-issue

224:        if (usdUpdateProposal.proposer == address(0)) revert NoProposalError();	// @audit-issue

225:        if (usdUpdateProposal.approvals[msg.sender] == _usdProposalId)	// @audit-issue

227:        if (usdUpdateProposal.disapprovals[msg.sender] == _usdProposalId)	// @audit-issue

229:        if (usdUpdateProposal.proposedPrice != _price)	// @audit-issue

246:        if (_duration > configStorage.getUint(StorageKey.MaxLockDuration))	// @audit-issue

321:        if (_mainAccount != _lockRecipient) revert SubAccountCannotLockError();	// @audit-issue

322:        if (_player.registrationDate == 0) revert AccountNotRegisteredError();	// @audit-issue

325:            if (msg.value != _quantity) revert ETHValueIncorrectError();	// @audit-issue

327:            if (msg.value != 0) revert InvalidMessageValueError();	// @audit-issue

330:            if (allowance < _quantity) revert InsufficientAllowanceError();	// @audit-issue

356:            if (	// @audit-issue

366:                if (numberNFTs > type(uint16).max) revert TooManyNFTsError();	// @audit-issue

408:        if (lockedToken.quantity < _quantity)	// @audit-issue

410:        if (lockedToken.unlockTime > uint32(block.timestamp))	// @audit-issue
```
[48](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L48-L48), [55](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L55-L55), [101](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L101-L101), [106](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L106-L106), [119](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L119-L119), [133](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L133-L133), [157](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L157-L157), [159](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L159-L159), [191](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L191-L191), [192](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L192-L192), [194](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L194-L194), [196](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L196-L196), [224](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L224-L224), [225](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L225-L225), [227](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L227-L227), [229](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L229-L229), [246](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L246-L246), [321](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L321-L321), [322](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L322-L322), [325](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L325-L325), [327](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L327-L327), [330](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L330-L330), [356](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L356-L356), [366](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L366-L366), [408](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L408-L408), [410](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L410-L410), 


#### Recommendation

Adhere to the Solidity style guide by formatting single-line `if` statements with the body on the same line as the condition. For example, use `if (x < 10) x += 1;` for concise and clear code. This style of formatting enhances readability and maintains consistency with Solidity's recommended best practices. It's particularly effective for simple and short conditional operations within the contract's code.

### NatSpec: Contract declarations should have `@author` tag
In the world of decentralized code, giving credit is key. NatSpec's `@author` tag acknowledges the minds behind the code. It appears this Solidity contract omits the `@author` directive in its NatSpec annotations. Properly attributing code to its contributors not only recognizes effort but also aids in establishing trust and credibility. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/managers/LockManager.sol

14:contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {	// @audit-issue missing `@author` tag
```
[14](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L14-L14), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@dev` tag
NatSpec comments are a critical part of Solidity's documentation system, designed to help developers and others understand the behavior and purpose of a contract. The `@dev` tag, in particular, provides context and insight into the contract's development considerations. A missing `@dev` comment can lead to misunderstandings about the contract, making it harder for others to contribute to or use the contract effectively. Therefore, it's highly recommended to include `@dev` comments in the documentation to enhance code readability and maintainability. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/managers/LockManager.sol

14:contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {	// @audit-issue missing `@dev` tag
```
[14](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L14-L14), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@notice` tag
The `@notice` is used to explain to users what the contract does. The compiler interprets `///` or `/**` comments [as this tag](https://docs.soliditylang.org/en/latest/natspec-format.html#tags) if one wasn't explicitly provided.

```solidity
Path: ./src/managers/LockManager.sol

14:contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {	// @audit-issue missing `@notice` tag
```
[14](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L14-L14), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@title` tag
The `@title` is used to explain to users what the contract does. The compiler interprets `///` or `/**` comments [as this tag](https://docs.soliditylang.org/en/latest/natspec-format.html#tags) if one wasn't explicitly provided.

```solidity
Path: ./src/managers/LockManager.sol

14:contract LockManager is BaseBlastManager, ILockManager, ReentrancyGuard {	// @audit-issue missing `@title` tag
```
[14](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L14-L14), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Use `@inheritdoc` for overriden functions.
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including @param tags will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/managers/LockManager.sol

85:    function configUpdated() external override onlyConfigStorage {	// @audit-issue missing `@inheritdoc` tag
```
[85](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L85-L85), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function `@return` tag is missing
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including `@return` tag will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/managers/LockManager.sol

430:    function getLocked(	// @audit-issue missing `@return` tag

461:    function getLockedWeightedValue(	// @audit-issue missing `@return` tag

490:    function getConfiguredToken(	// @audit-issue missing `@return` tag

496:    function getPlayerSettings(	// @audit-issue missing `@return` tag
```
[430](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L430-L430), [461](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L461-L461), [490](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L490-L490), [496](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L496-L496), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function declarations should have `@notice` tag
The `@notice` tag in NatSpec comments is used to provide important explanations to end users about what a function does. It appears that this contract's function declarations are missing `@notice` tags in their NatSpec annotations.

The absence of `@notice` tags reduces the contract's transparency and could lead to misunderstandings about a function's purpose and behavior.  [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/managers/LockManager.sol

60:    constructor(address _configStorage) {	// @audit-issue missing `@notice` tag

65:    function _reconfigure() internal {	// @audit-issue missing `@notice` tag

85:    function configUpdated() external override onlyConfigStorage {	// @audit-issue missing `@notice` tag

89:    fallback() external payable {	// @audit-issue missing `@notice` tag

93:    receive() external payable {	// @audit-issue missing `@notice` tag

98:    function configureLockdrop(	// @audit-issue missing `@notice` tag

115:    function configureToken(	// @audit-issue missing `@notice` tag

129:    function setUSDThresholds(	// @audit-issue missing `@notice` tag

142:    function proposeUSDPrice(	// @audit-issue missing `@notice` tag

177:    function approveUSDPrice(	// @audit-issue missing `@notice` tag

210:    function disapproveUSDPrice(	// @audit-issue missing `@notice` tag

245:    function setLockDuration(uint256 _duration) external notPaused {	// @audit-issue missing `@notice` tag

275:    function lockOnBehalf(	// @audit-issue missing `@notice` tag

297:    function lock(	// @audit-issue missing `@notice` tag

311:    function _lock(	// @audit-issue missing `@notice` tag

401:    function unlock(	// @audit-issue missing `@notice` tag

430:    function getLocked(	// @audit-issue missing `@notice` tag

461:    function getLockedWeightedValue(	// @audit-issue missing `@notice` tag

490:    function getConfiguredToken(	// @audit-issue missing `@notice` tag

496:    function getPlayerSettings(	// @audit-issue missing `@notice` tag

506:    function _execUSDPriceUpdate() internal {	// @audit-issue missing `@notice` tag
```
[60](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L60-L60), [65](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L65-L65), [85](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L85-L85), [89](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L89-L89), [93](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L93-L93), [98](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L98-L98), [115](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L115-L115), [129](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L129-L129), [142](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L142-L142), [177](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L177-L177), [210](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L210-L210), [245](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L245-L245), [275](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L275-L275), [297](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L297-L297), [311](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L311-L311), [401](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L401-L401), [430](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L430-L430), [461](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L461-L461), [490](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L490-L490), [496](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L496-L496), [506](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L506-L506), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function declarations should have `@dev` tag
Some functions have an incomplete NatSpec: add a `@dev` notation to describe the function to improve the code documentation.

```solidity
Path: ./src/managers/LockManager.sol

60:    constructor(address _configStorage) {	// @audit-issue missing `@dev` tag

65:    function _reconfigure() internal {	// @audit-issue missing `@dev` tag

85:    function configUpdated() external override onlyConfigStorage {	// @audit-issue missing `@dev` tag

89:    fallback() external payable {	// @audit-issue missing `@dev` tag

93:    receive() external payable {	// @audit-issue missing `@dev` tag

98:    function configureLockdrop(	// @audit-issue missing `@dev` tag

115:    function configureToken(	// @audit-issue missing `@dev` tag

129:    function setUSDThresholds(	// @audit-issue missing `@dev` tag

142:    function proposeUSDPrice(	// @audit-issue missing `@dev` tag

177:    function approveUSDPrice(	// @audit-issue missing `@dev` tag

210:    function disapproveUSDPrice(	// @audit-issue missing `@dev` tag

245:    function setLockDuration(uint256 _duration) external notPaused {	// @audit-issue missing `@dev` tag

275:    function lockOnBehalf(	// @audit-issue missing `@dev` tag

297:    function lock(	// @audit-issue missing `@dev` tag

311:    function _lock(	// @audit-issue missing `@dev` tag

401:    function unlock(	// @audit-issue missing `@dev` tag

430:    function getLocked(	// @audit-issue missing `@dev` tag

461:    function getLockedWeightedValue(	// @audit-issue missing `@dev` tag

490:    function getConfiguredToken(	// @audit-issue missing `@dev` tag

496:    function getPlayerSettings(	// @audit-issue missing `@dev` tag

506:    function _execUSDPriceUpdate() internal {	// @audit-issue missing `@dev` tag
```
[60](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L60-L60), [65](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L65-L65), [85](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L85-L85), [89](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L89-L89), [93](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L93-L93), [98](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L98-L98), [115](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L115-L115), [129](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L129-L129), [142](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L142-L142), [177](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L177-L177), [210](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L210-L210), [245](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L245-L245), [275](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L275-L275), [297](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L297-L297), [311](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L311-L311), [401](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L401-L401), [430](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L430-L430), [461](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L461-L461), [490](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L490-L490), [496](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L496-L496), [506](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L506-L506), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function/Constructor `@param` tag is missing
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including @param tags will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/managers/LockManager.sol

60:    constructor(address _configStorage) {	// @audit-issue missing `@param` tag

98:    function configureLockdrop(	// @audit-issue missing `@param` tag

115:    function configureToken(	// @audit-issue missing `@param` tag

129:    function setUSDThresholds(	// @audit-issue missing `@param` tag

142:    function proposeUSDPrice(	// @audit-issue missing `@param` tag

177:    function approveUSDPrice(	// @audit-issue missing `@param` tag

210:    function disapproveUSDPrice(	// @audit-issue missing `@param` tag

245:    function setLockDuration(uint256 _duration) external notPaused {	// @audit-issue missing `@param` tag

275:    function lockOnBehalf(	// @audit-issue missing `@param` tag

297:    function lock(	// @audit-issue missing `@param` tag

311:    function _lock(	// @audit-issue missing `@param` tag

401:    function unlock(	// @audit-issue missing `@param` tag

430:    function getLocked(	// @audit-issue missing `@param` tag

461:    function getLockedWeightedValue(	// @audit-issue missing `@param` tag

490:    function getConfiguredToken(	// @audit-issue missing `@param` tag

496:    function getPlayerSettings(	// @audit-issue missing `@param` tag
```
[60](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L60-L60), [98](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L98-L98), [115](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L115-L115), [129](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L129-L129), [142](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L142-L142), [177](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L177-L177), [210](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L210-L210), [245](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L245-L245), [275](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L275-L275), [297](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L297-L297), [311](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L311-L311), [401](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L401-L401), [430](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L430-L430), [461](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L461-L461), [490](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L490-L490), [496](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L496-L496), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Modifier declarations should have `@dev` tag
Some modifiers have an incomplete NatSpec: add a `@dev` notation to describe the modifier to improve the code documentation.

```solidity
Path: ./src/managers/LockManager.sol

47:    modifier onlyConfiguredToken(address _tokenContract) {	// @audit-issue missing `@dev` tag

54:    modifier onlyActiveToken(address _tokenContract) {	// @audit-issue missing `@dev` tag
```
[47](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L47-L47), [54](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L54-L54), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Modifiers `@param` tag is missing
Function modifiers should include NatSpec comments with @param tags describing each input parameter. This promotes better code readability and documentation. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/managers/LockManager.sol

47:    modifier onlyConfiguredToken(address _tokenContract) {	// @audit-issue missing `@param` tag

54:    modifier onlyActiveToken(address _tokenContract) {	// @audit-issue missing `@param` tag
```
[47](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L47-L47), [54](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/./src/managers/LockManager.sol#L54-L54), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).
