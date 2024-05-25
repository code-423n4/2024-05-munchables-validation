
https://github.com/code-423n4/2024-05-munchables/blob/main/src/managers/LockManager.sol#L166
# L1. Using uint32 for block.timestamp is generally not sufficient for representing block timestamps

## Impact
This range covers timestamps up to around the year 2106.

## Proof of Concept
```solidity
File: src/interfaces/ILockManager.sol#L61
    struct USDUpdateProposal {
        uint32 proposedDate; // @audit around year 2106?
        address proposer;
        address[] contracts;
        uint256 proposedPrice;
        mapping(address => uint32) approvals;
        mapping(address => uint32) disapprovals;
        uint8 approvalsCount;
        uint8 disapprovalsCount;
    }
File: src/managers/LockManager.sol#L104
    uint32(block.timestamp)
File: src/managers/LockManager.sol#L166
    usdUpdateProposal.proposedDate = uint32(block.timestamp);
File: src/managers/LockManager.sol#L249
    playerSettings[msg.sender].lockDuration = uint32(_duration);
File: src/managers/LockManager.sol#L257
    uint32(block.timestamp) + uint32(_duration) <
File: src/managers/LockManager.sol#L267
    uint32(_duration);
File: src/managers/LockManager.sol#L353
    lockdrop.start <= uint32(block.timestamp) &&
File: src/managers/LockManager.sol#L354
    lockdrop.end >= uint32(block.timestamp)
File: src/managers/LockManager.sol#L359
    uint32(configStorage.getUint(StorageKey.MaxLockDuration))
File: src/managers/LockManager.sol#L381
    lockedToken.lastLockTime = uint32(block.timestamp);
File: src/managers/LockManager.sol#L383
    uint32(block.timestamp) +
File: src/managers/LockManager.sol#L384
    uint32(_lockDuration);
File: src/managers/LockManager.sol#L410
    if (lockedToken.unlockTime > uint32(block.timestamp))
```

## Tools Used
Manual review

## Recommended Mitigation Steps
It's recommended to use uint48 for timestamps.




# NC1. Event missing indexed field

Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. This is especially useful when it comes to filtering based on an address. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Where applicable, each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three applicable fields, all of the applicable fields should be indexed.

```solidity
File: src/interfaces/ILockManager.sol
159:     event TokenConfigured(address _tokenContract, ConfiguredToken _tokenData); 
168:     event ProposedUSDPrice(address _proposer, uint256 _price);
172:     event ApprovedUSDPrice(address _approver);
176:     event DisapprovedUSDPrice(address _disapprover);
225:     event USDPriceUpdated(address _tokenContract, uint256 _newPrice);
```

# NC2. Some errors are not used

Remove the unused error types.

```solidity
File: src/interfaces/ILockManager.sol
228:     error OnlyAccountManagerError();
245:     error USDPriceInvalidError();
293:     error InvalidTokenContractError();

```

# NC3. Some events are not used
Remove the unused event types.

```solidity
File: src/interfaces/ILockManager.sol
220:     event DiscountFactorUpdated(uint256 discountFactor);
```

