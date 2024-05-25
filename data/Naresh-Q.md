## Summary

| |Issue|Instances| 
|-|:-|:-:|
| [[L-01](#l-01)] | Large transfers may not work with some `ERC20` tokens | 2| 
| [[L-02](#l-02)] | Consider adding validation of user inputs | 4| 
| [[L-03](#l-03)] | Array is `push()`ed but not `pop()`ed | 1| 
| [[L-04](#l-04)] | Privileged functions can create points of failure | 9| 
| [[L-05](#l-05)] | Code does not follow the best practice of check-effects-interaction | 1| 
| [[L-06](#l-06)] | Possible reentrancy with callback on transfer tokens | 1| 
| [[L-07](#l-07)] | Events may be emitted out of order due to reentrancy | 2| 
| [[L-08](#l-08)] | Functions calling contracts/addresses with transfer hooks are missing reentrancy guards | 1| 
| [[L-09](#l-09)] | Missing zero address check in constructor | 1| 
| [[L-10](#l-10)] | The `nonReentrant` modifier should be placed first in a function declaration | 3| 
| [[L-11](#l-11)] | State variables not capped at reasonable values | 2| 
| [[L-12](#l-12)] | Consider implementing two-step procedure for updating protocol addresses | 2| 

### Low Risk Issues

### [L-01]<a name="l-01"></a> Large transfers may not work with some `ERC20` tokens

Some `IERC20` implementations (e.g `UNI`, `COMP`) may fail if the valued transferred is larger than `uint96`. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)

*There are 2 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

376:             token.transferFrom(_tokenOwner, address(this), _quantity);

423:             token.transfer(msg.sender, _quantity);

```


*GitHub* : [376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L376-L376), [423](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L423-L423)

### [L-02]<a name="l-02"></a> Consider adding validation of user inputs

There are no validations done on the arguments below. Consider that the Solidity [documentation](https://docs.soliditylang.org/en/latest/control-structures.html#panic-via-assert-and-error-via-require) states that `Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix`. This means that there should be explicit checks for expected ranges of inputs. Underflows/overflows result in panics should not be used as range checks, and allowing funds to be sent to `0x0`, which is the default value of address variables and has many gotchas, should be avoided.

*There are 4 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

// @audit missing checks for -->  _tokenContract
115:     function configureToken(
116:         address _tokenContract,
117:         ConfiguredToken memory _tokenData
118:     ) external onlyAdmin {

// @audit missing checks for -->  _contracts
142:     function proposeUSDPrice(
143:         uint256 _price,
144:         address[] calldata _contracts
145:     )
146:         external
147:         onlyOneOfRoles(
148:             [
149:                 Role.PriceFeed_1,
150:                 Role.PriceFeed_2,
151:                 Role.PriceFeed_3,
152:                 Role.PriceFeed_4,
153:                 Role.PriceFeed_5
154:             ]
155:         )
156:     {

// @audit missing checks for -->  _tokenContract
275:     function lockOnBehalf(
276:         address _tokenContract,
277:         uint256 _quantity,
278:         address _onBehalfOf
279:     )
280:         external
281:         payable
282:         notPaused
283:         onlyActiveToken(_tokenContract)
284:         onlyConfiguredToken(_tokenContract)
285:         nonReentrant
286:     {

// @audit missing checks for -->  _tokenContract
297:     function lock(
298:         address _tokenContract,
299:         uint256 _quantity
300:     )
301:         external
302:         payable
303:         notPaused
304:         onlyActiveToken(_tokenContract)
305:         onlyConfiguredToken(_tokenContract)
306:         nonReentrant
307:     {

```


*GitHub* : [115](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L115-L118), [142](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L142-L156), [275](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L275-L286), [297](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L297-L307)

### [L-03]<a name="l-03"></a> Array is `push()`ed but not `pop()`ed

Array entries are added but are never removed. Consider whether this should be the case, or whether there should be a maximum, or whether old entries should be removed. Cases where there are specific potential problems will be flagged separately under a different issue.

*There are 1 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

122:             configuredTokenContracts.push(_tokenContract);

```


*GitHub* : [122](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L122-L122)

### [L-04]<a name="l-04"></a> Privileged functions can create points of failure

Ensure such accounts are protected and consider implementing multi sig to prevent a single point of failure

*There are 9 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

85:     function configUpdated() external override onlyConfigStorage {

98:     function configureLockdrop(
99:         Lockdrop calldata _lockdropData
100:     ) external onlyAdmin {

115:     function configureToken(
116:         address _tokenContract,
117:         ConfiguredToken memory _tokenData
118:     ) external onlyAdmin {

129:     function setUSDThresholds(
130:         uint8 _approve,
131:         uint8 _disapprove
132:     ) external onlyAdmin {

142:     function proposeUSDPrice(
143:         uint256 _price,
144:         address[] calldata _contracts
145:     )
146:         external
147:         onlyOneOfRoles(
148:             [
149:                 Role.PriceFeed_1,
150:                 Role.PriceFeed_2,
151:                 Role.PriceFeed_3,
152:                 Role.PriceFeed_4,
153:                 Role.PriceFeed_5
154:             ]
155:         )
156:     {

177:     function approveUSDPrice(
178:         uint256 _price
179:     )
180:         external
181:         onlyOneOfRoles(
182:             [
183:                 Role.PriceFeed_1,
184:                 Role.PriceFeed_2,
185:                 Role.PriceFeed_3,
186:                 Role.PriceFeed_4,
187:                 Role.PriceFeed_5
188:             ]
189:         )
190:     {

210:     function disapproveUSDPrice(
211:         uint256 _price
212:     )
213:         external
214:         onlyOneOfRoles(
215:             [
216:                 Role.PriceFeed_1,
217:                 Role.PriceFeed_2,
218:                 Role.PriceFeed_3,
219:                 Role.PriceFeed_4,
220:                 Role.PriceFeed_5
221:             ]
222:         )
223:     {

275:     function lockOnBehalf(
276:         address _tokenContract,
277:         uint256 _quantity,
278:         address _onBehalfOf
279:     )
280:         external
281:         payable
282:         notPaused
283:         onlyActiveToken(_tokenContract)
284:         onlyConfiguredToken(_tokenContract)
285:         nonReentrant
286:     {

297:     function lock(
298:         address _tokenContract,
299:         uint256 _quantity
300:     )
301:         external
302:         payable
303:         notPaused
304:         onlyActiveToken(_tokenContract)
305:         onlyConfiguredToken(_tokenContract)
306:         nonReentrant
307:     {

```


*GitHub* : [85](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L85-L85), [98](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L98-L100), [115](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L115-L118), [129](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L129-L132), [142](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L142-L156), [177](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L177-L190), [210](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L210-L223), [275](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L275-L286), [297](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L297-L307)

### [L-05]<a name="l-05"></a> Code does not follow the best practice of check-effects-interaction

Code should follow the best-practice of [CEI](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs

*There are 1 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

// @audit token.transferFrom() called on line 376 
387:         playerSettings[_lockRecipient].lockDuration = _lockDuration;

```


*GitHub* : [387](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L387-L387)

### [L-06]<a name="l-06"></a> Possible reentrancy with callback on transfer tokens

The following functions don't apply the [CEI](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/) pattern. It's possible to reenter after the transfer if the token has some kind of callback functionality (e.g. ERC777/ERC1155).

*There are 1 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

// @audit state update on line 387
376:             token.transferFrom(_tokenOwner, address(this), _quantity);

```


*GitHub* : [376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L376-L376)

### [L-07]<a name="l-07"></a> Events may be emitted out of order due to reentrancy

If a reentrancy occurs, some events may be emitted in an unexpected order, and this may be a problem if a third party expects a specific order for these events. Ensure that events are emitted before external calls and follow the best practice of CEI.

*There are 2 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

389:         emit Locked(
390:             _lockRecipient,
391:             _tokenOwner,
392:             _tokenContract,
393:             _quantity,
394:             remainder,
395:             numberNFTs,
396:             _lockDuration
397:         );

426:         emit Unlocked(msg.sender, _tokenContract, _quantity);

```


*GitHub* : [389](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L389-L397), [426](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L426-L426)

### [L-08]<a name="l-08"></a> Functions calling contracts/addresses with transfer hooks are missing reentrancy guards

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to [read-only reentrancies](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/) with no way to protect against it, except by block-listing the whole protocol.

*There are 1 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

// @audit function '_lock()' is missing Reentrancy guard
376:             token.transferFrom(_tokenOwner, address(this), _quantity);

```


*GitHub* : [376](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L376-L376)

### [L-09]<a name="l-09"></a> Missing zero address check in constructor

Constructors often take address parameters to initialize important components of a contract, such as owner or linked contracts. However, without a checking, there's a risk that an address parameter could be mistakenly set to the zero address `(0x0)`. This could be due to an error or oversight during contract deployment. A zero address in a crucial role can cause serious issues, as it cannot perform actions like a normal address, and any funds sent to it will be irretrievable. It's therefore crucial to include a zero address check in constructors to prevent such potential problems. If a zero address is detected, the constructor should revert the transaction.

*There are 1 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

// @audit missing checks for -->  _configStorage
60:     constructor(address _configStorage) {

```


*GitHub* : [60](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L60-L60)

### [L-10]<a name="l-10"></a> The `nonReentrant` modifier should be placed first in a function declaration

In functions with multiple modifiers, including `nonReentrant`, the `nonReentrant` modifier should be placed first in the ordering to prevent reentrancy in prior modifiers.

*There are 3 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

275:     function lockOnBehalf(
276:         address _tokenContract,
277:         uint256 _quantity,
278:         address _onBehalfOf
279:     )
280:         external
281:         payable
282:         notPaused
283:         onlyActiveToken(_tokenContract)
284:         onlyConfiguredToken(_tokenContract)
285:         nonReentrant
286:     {
287:         address tokenOwner = msg.sender;
288:         address lockRecipient = msg.sender;
289:         if (_onBehalfOf != address(0)) {
290:             lockRecipient = _onBehalfOf;
291:         }
292: 
293:         _lock(_tokenContract, _quantity, tokenOwner, lockRecipient);
294:     }

297:     function lock(
298:         address _tokenContract,
299:         uint256 _quantity
300:     )
301:         external
302:         payable
303:         notPaused
304:         onlyActiveToken(_tokenContract)
305:         onlyConfiguredToken(_tokenContract)
306:         nonReentrant
307:     {
308:         _lock(_tokenContract, _quantity, msg.sender, msg.sender);
309:     }

401:     function unlock(
402:         address _tokenContract,
403:         uint256 _quantity
404:     ) external notPaused nonReentrant {
405:         LockedToken storage lockedToken = lockedTokens[msg.sender][
406:             _tokenContract
407:         ];
408:         if (lockedToken.quantity < _quantity)
409:             revert InsufficientLockAmountError();
410:         if (lockedToken.unlockTime > uint32(block.timestamp))
411:             revert TokenStillLockedError();
412: 
413:         // force harvest to make sure that they get the schnibbles that they are entitled to
414:         accountManager.forceHarvest(msg.sender);
415: 
416:         lockedToken.quantity -= _quantity;
417: 
418:         // send token
419:         if (_tokenContract == address(0)) {
420:             payable(msg.sender).transfer(_quantity);
421:         } else {
422:             IERC20 token = IERC20(_tokenContract);
423:             token.transfer(msg.sender, _quantity);
424:         }
425: 
426:         emit Unlocked(msg.sender, _tokenContract, _quantity);
427:     }

```


*GitHub* : [275](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L275-L294), [297](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L297-L309), [401](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L401-L427)

### [L-11]<a name="l-11"></a> State variables not capped at reasonable values

Consider adding minimum/maximum value checks to ensure that the state variables below can never be used to excessively harm users, including via griefing

*There are 2 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

// @audit _approve
135:         APPROVE_THRESHOLD = _approve;

// @audit _disapprove
136:         DISAPPROVE_THRESHOLD = _disapprove;

```


*GitHub* : [135](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L135-L135), [136](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L136-L136)

### [L-12]<a name="l-12"></a> Consider implementing two-step procedure for updating protocol addresses

A copy-paste error or a typo may end up bricking protocol functionality, or sending tokens to an address with no known private key. Consider implementing a two-step procedure for updating protocol addresses, where the recipient is set as pending, and must 'accept' the assignment by making an affirmative call. A straight forward way of doing this would be to have the target contracts implement [EIP-165](https://eips.ethereum.org/EIPS/eip-165), and to have the 'set' functions ensure that the recipient is of the right interface type.

*There are 2 instance(s) of this issue:*

```solidity
📁 File: src/managers/LockManager.sol

115:     function configureToken(
116:         address _tokenContract,
117:         ConfiguredToken memory _tokenData
118:     ) external onlyAdmin {

275:     function lockOnBehalf(
276:         address _tokenContract,
277:         uint256 _quantity,
278:         address _onBehalfOf
279:     )
280:         external
281:         payable
282:         notPaused
283:         onlyActiveToken(_tokenContract)
284:         onlyConfiguredToken(_tokenContract)
285:         nonReentrant
286:     {

```


*GitHub* : [115](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L115-L118), [275](https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L275-L286)
