## Summary
In the `LockManager::getConfiguredToken` and `LockManager::getPlayerSettings` functions, the contract fails to verify the existence of the records being queried. This oversight can lead to unintended behavior, potential errors, and security risks. To mitigate these issues, it is recommended to add checks ensuring that the records exist before accessing them. This will enhance the contract's reliability and security.

## Vulnerability Detail
The functions `LockManager::getConfiguredToken` and `LockManager::getPlayerSettings` do not check if the queried records exist. As a result, they might attempt to access data for non-existent or unconfigured tokens and players.

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L490-L494

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L496-L500

## Impact
- **Unintended Behavior:** The functions might return incorrect or default values if the records do not exist.
- **Potential Errors:** Attempting to access non-existent records can lead to errors or undefined behavior.
- **Security Risks:** The lack of validation can be exploited by attackers to manipulate the contract's behavior or extract unintended information

## Code Snippet
```solidity
function getConfiguredToken(address _tokenContract) external view returns (ConfiguredToken memory _token) {

    _token = configuredTokens[_tokenContract]; 

}

function getPlayerSettings(address _player) external view returns (PlayerSettings memory _settings) {

    _settings = playerSettings[_player];

}
```

## Tool used
Manual Review

## Recommendation
Implement checks in both of these functions.