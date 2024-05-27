Executive summary
%%{init: {"theme": "base", "themeVariables": {"pie1": "#00FF00","pie2": "#FFFF00","pie3": "#FFA500","pie4": "#FFFFFF"}}}%%
pie showData 
    title Risks
   "Medium" : 3
   "Low" : 9
   "Quality" : 2
   "Gas" : 24

Contents
Medium risks
Total 3 instances over 1 risk<br>

ID	Risk	Instances	Gas Savings
[DOW-UUTIERC20]	Unsafe use of transfer()/transferFrom() with IERC20	3	0
Low risks
Total 9 instances over 1 risk<br>

ID	Risk	Instances	Gas Savings
[SWC-120]	Weak sources of randomness from chain attributes	9	0
Non-Critical/Quality risks
Total 2 instances over 1 risk<br>

ID	Risk	Instances	Gas Savings
[DOW-UCO]	Constants should be defined rather than using magic numbers	2	0
Gas risks
Total 24 instances over 7 risks<br>

ID	Risk	Instances	Gas Savings
[GAS-MEM]	Use calldata instead of memory for immutable arguments	7	7
[GAS-UIC]	Use != 0 instead of > 0 for unsigned integer comparison	2	12
[GAS-PIN]	Pre-increments and pre-decrements are cheaper than post-increments and post-decrements	7	35
[DOW-CAL]	a = a + b is more gas effective than a += b for state variables (excluding arrays and mappings)	1	3
[DOW-APE]	a = a + b is more gas effective than a += b for state variables (excluding arrays and mappings)	2	32
[GAS-PAF]	Functions guaranteed to revert when called by normal users can be marked payable	1	1
[GAS-IUL]	Increments can be unchecked in for-loops	4	140
Details contents
Medium risks
[DOW-UUTIERC20]<a name="DOW-UUTIERC20"> Unsafe use of transfer()/transferFrom() with IERC20
Description:<br> Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions on L1 do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert (see this link for a test case).<br> <br>Recommendation:<br> Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead<br> There are 3 instances of this risk: <details><summary>see instances</summary>File: LockManager.sol

376:             token.transferFrom(_tokenOwner, address(this), _quantity);
420:             payable(msg.sender).transfer(_quantity);
423:             token.transfer(msg.sender, _quantity);

GitHub : L376 L420 L423

<br> </details>

Low risks
[SWC-120]<a name="SWC-120"> Weak sources of randomness from chain attributes
Description:<br>Creating a strong enough source of randomness in Ethereum is very challenging. For example, use of block.timestamp is insecure, as a miner can choose to provide any timestamp within a few seconds and still get his block accepted by others. Use of blockhash, block.difficulty and other fields is also insecure, as they're controlled by the miner. If the stakes are high, the miner can mine lots of blocks in a short time by renting hardware, pick the block that has required block hash for him to win, and drop all others.<br>https://swcregistry.io/docs/SWC-120 <br>Recommendation:<br>Using commitment scheme, e.g. RANDAO.<br>Using external sources of randomness via oracles, e.g. Oraclize. Note that this approach requires trusting in oracle, thus it may be reasonable to use multiple oracles. <br>Using Bitcoin block hashes, as they are more expensive to mine.<br> There are 9 instances of this risk: <details><summary>see instances</summary>File: LockManager.sol

101: block.timestamp
104: block.timestamp
166: block.timestamp
257: block.timestamp
353: block.timestamp
354: block.timestamp
381: block.timestamp
383: block.timestamp
410: block.timestamp

GitHub : L101 L104 L166 L257 L353 L354 L381 L383 L410

<br> </details>

Non-Critical/Quality risks
[DOW-UCO]<a name="DOW-UCO"> Constants should be defined rather than using magic numbers
Description:<br> Constants should be defined rather than using magic numbers<br> <br>Recommendation:<br> cf. Description<br> <br> There are 2 instances of this risk: <details><summary>see instances</summary>File: LockManager.sol

472:                 // We are assuming all tokens have a maximum of 18
474:                     (18

GitHub : L472 L474

<br> </details>

Gas risks
[GAS-MEM]<a name="GAS-MEM"> Use calldata instead of memory for immutable arguments
Description:<br> If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.<br> <br><br>Recommendation:<br> Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory.<br> There are 7 instances of this risk: <details><summary>see instances</summary>File: LockManager.sol

117:         ConfiguredToken memory _tokenData
319:             MunchablesCommonLib.Player memory _player
432:     ) external view returns (LockedTokenWithMetadata[] memory _lockedTokens) {
435:             memory tmpLockedTokens = new LockedTokenWithMetadata[](
439:             LockedToken memory tmpLockedToken;
492:     ) external view returns (ConfiguredToken memory _token) {
498:     ) external view returns (PlayerSettings memory _settings) {

GitHub : L117 L319 L432 L435 L439 L492 L498

<br> </details>

[GAS-UIC]<a name="GAS-UIC"> Use != 0 instead of > 0 for unsigned integer comparison
Description:<br> != 0 costs less gas than > 0 <br> See for more information:<br> Source1 <br> Source2 <br> <br>Recommendation:<br> Use != 0 instead of > 0 for unsigned integer comparison in:<br> <br> There are 2 instances of this risk: <details><summary>see instances</summary>File: LockManager.sol

254: > 0
468: >
                0

GitHub : L254 L468-L469

<br> </details>

[GAS-PIN]<a name="GAS-PIN"> Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
Description:<br> ++i costs less gas than i++, especially when it's used in loops (i-- too). See for more information: https://gist.github.com/ahmedovv123/ac550b389bcbe21043c613e6c6c1b563#GAS-5 <br> <br>Recommendation:<br> Use ++i or --i in: <br> <br> There are 7 instances of this risk: <details><summary>see instances</summary>File: LockManager.sol

171: usdUpdateProposal.approvalsCount++
200: usdUpdateProposal.approvalsCount++
232: usdUpdateProposal.disapprovalsCount++
252: i++
438: i++
466: i++
512: i++

GitHub : L171 L200 L232 L252 L438 L466 L512

<br> </details>

[DOW-CAL]<a name="DOW-CAL"> a = a + b is more gas effective than a += b for state variables (excluding arrays and mappings)
Description:<br> If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).<br> <br>Recommendation:<br> Better use a = a + b than a += b<br> There is 1 instance of this risk: <details><summary>see instance</summary>File: LockManager.sol

380:         lockedToken.quantity += _quantity;

GitHub : L380

<br> </details>

[DOW-APE]<a name="DOW-APE"> a = a + b is more gas effective than a += b for state variables (excluding arrays and mappings)
Description:<br> a = a + b is more gas effective than a += b for state variables (excluding arrays and mappings)<br> <br>Recommendation:<br> This saves 16 gas per instance. and effectively send transactions from any address.<br> There are 2 instances of this risk: <details><summary>see instances</summary>File: LockManager.sol

380:         lockedToken.quantity += _quantity;
476:                 lockedWeighted +=
                    (deltaDecimal *

GitHub : L380 L476-L477

<br> </details>

[GAS-PAF]<a name="GAS-PAF"> Functions guaranteed to revert when called by normal users can be marked payable
Description:<br> If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. <br> <br><br>Recommendation:<br> Use of payable functions: Payable functions can be slightly more gas-efficient than non-payable ones. This is because the compiler doesn’t need to check for the transfer of Ether in payable functions.<br> <br> There is 1 instance of this risk: <details><summary>see instance</summary>File: LockManager.sol

85: function configUpdated() external override only

GitHub : L85

<br> </details>

[GAS-IUL]<a name="GAS-IUL"> Increments can be unchecked in for-loops
Description:<br> Use ++i or --i in solidity version 0.8.0, should be unchecked{++i}/unchecked{--i} when it is not possible for them to overflow, as is the case when used in FOR and WHILE loops The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP <br> <br>Recommendation:<br> Use *unchecked{++i/i++}* or *unchecked{--i/i--}*<br> <br> <br> There are 4 instances of this risk: <details><summary>see instances</summary>File: LockManager.sol

252: for (uint256 i; i < configuredTokensLength; i++)
438: for (uint256 i; i < configuredTokensLength; i++)
466: for (uint256 i; i < configuredTokensLength; i++)
512: for (uint256 i; i < updateTokensLength; i++)

GitHub : L252 L438 L466 L512

<br> </details>