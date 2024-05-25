## Threshold Setting Without Limits

The `setUSDThresholds` function lacks validation for the lower boundaries of `_approve` and `_disapprove` values.

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L129

```solidity

    function setUSDThresholds(
        uint8 _approve,
        uint8 _disapprove
    ) external onlyAdmin {//TODO no low boundary limit for `_approve` and `_disapprove`
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
        APPROVE_THRESHOLD = _approve;
        DISAPPROVE_THRESHOLD = _disapprove;

        emit USDThresholdUpdated(_approve, _disapprove);
    }

```

Setting these thresholds too low could allow critical decisions to be approved/disapproved with minimal consensus,
undermining the security of the decision-making process.

To resolve this issue, the `setUSDThresholds()` function should introduce minimum limits for approval and disapproval
thresholds to ensure adequate participation decisions.

### Token Decimal Consistency Check

In the `configureToken` function, it does not verify if the `_tokenData.decimals` property of a token being configured
matches the actual decimals used by the token contract.

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L115

```solidity

    function configureToken(
        address _tokenContract,
        ConfiguredToken memory _tokenData
    ) external onlyAdmin {
        if (_tokenData.nftCost == 0) revert NFTCostInvalidError();
        if (configuredTokens[_tokenContract].nftCost == 0) {
            // new token
            configuredTokenContracts.push(_tokenContract);
        }
        configuredTokens[_tokenContract] = _tokenData;

        emit TokenConfigured(_tokenContract, _tokenData);
    }

```

Misconfiguration can lead to incorrect calculations and potentially exploitable conditions if the token's actual
decimals differ from what the contract expects.

To address this issue, it should implement a check to verify token decimals during the configuration to ensure
consistency with the token contract



### Potential Overflow Due to Not Check Existing NFT Numbers in `_lock`

The `_lock` checks if the number of NFTs (`numberNFTs`) calculated for the current locking transaction exceeds the
maximum value that a `uint16` can store (`65535`). However, this check does not consider any NFTs that the
user (`_lockRecipient`) may have previously accumulated and which have not yet been revealed or claimed.

```solidity=366
        if (numberNFTs > type(uint16).max) revert TooManyNFTsError();
```

https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L366

To resolve this issue, the `_lock` should include a check that considers both the NFTs claimable from the current
transaction and those that the user has already accrued but not yet claimed. This can be done by adding the number of
unrevealed NFTs (queried from the `nftOverlord` contract) to the `numberNFTs` before performing the check
against `uint16.max`. The corrected line of code would look like this:

```solidity=366
    uint16 existingNFTs = nftOverlord.getUnrevealedNFTs(_lockRecipient);
    if (numberNFTs + existingNFTs > type(uint16).max) revert TooManyNFTsError();
```