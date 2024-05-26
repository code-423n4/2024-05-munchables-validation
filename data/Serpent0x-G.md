## Issue: Direct initialization of variables with default values.
Line: `uint256 lockedWeighted = 0;`  #464 
Explanation: Initializing variables with default values explicitly costs unnecessary gas.
Mitigation: Remove explicit initialization of variables with default values.

#####################################################################################

## Issue: Repeatedly accessing array length inside loops.
Lines:
`if (_contracts.length == 0) revert ProposalInvalidContractsError();` #159
`uint256 configuredTokensLength = configuredTokenContracts.length;`   #251
`uint256 configuredTokensLength = configuredTokenContracts.length;`   #433
`uint256 configuredTokensLength = configuredTokenContracts.length;`   #465
`uint256 updateTokensLength = usdUpdateProposal.contracts.length;`    #511
Explanation: Caching array length outside loops saves gas by avoiding repeated access.
Mitigation: Cache array length outside loops to optimize gas usage.

#####################################################################################

## Issue: Using > for unsigned integer comparison.
Line: `if (lockedTokens[msg.sender][tokenContract].quantity > 0) {`   #254
Explanation: Comparisons with != 0 are cheaper than with > 0 for unsigned integers.
Mitigation: Replace > with != 0 for unsigned integer comparisons.


#####################################################################################

## Issue: Long revert strings in imports.
Lines:
`import "@openzeppelin/contracts/token/ERC20/ERC20.sol";`   #4 #5 #8 #9 #11 #105    
Other similar import lines
Explanation: Shortening revert strings to fit in 32 bytes decreases gas costs for deployment and revert conditions.
Mitigation: Shorten revert strings or consider using custom errors for efficiency.

