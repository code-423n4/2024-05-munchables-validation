# LOW-1: USDPrice must never be able be set to 0

Links to affected code:
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L142
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L177

## Impact

**Likelihood: LOW**
Price can only be set by one of the trusted roles and must be approved by a second address (also trusted).

**Impact: HIGH**

Setting the USD price to 0 in the protocol would have significant consequences for both the protocol and its users. Based on the ```LockManager.sol``` code, the USD price is a critical component in determining the cost of NFTs in terms of USD.

**NFT Acquisition Cost Becomes Zero:** If the USD price is set to 0, it implies that the cost to acquire NFTs (which are priced in USD within the system) would effectively become zero. This could lead to a massive influx of NFT claims, as users take advantage of the zero cost.

## Proof of Concept
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L142

No checks are done in the ```LockManager.sol::proposeUSDPrice``` function if ```_price``` is equal to 0.

```solidity
/// @inheritdoc ILockManager
    function proposeUSDPrice(
        uint256 _price,
        address[] calldata _contracts
    )
        external
        onlyOneOfRoles(
            [
                Role.PriceFeed_1,
                Role.PriceFeed_2,
                Role.PriceFeed_3,
                Role.PriceFeed_4,
                Role.PriceFeed_5
            ]
        )
    {
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
        if (_contracts.length == 0) revert ProposalInvalidContractsError();
        delete usdUpdateProposal;

        // Approvals will use this because when the struct is deleted the approvals remain
        ++_usdProposalId;

        usdUpdateProposal.proposedDate = uint32(block.timestamp);
        usdUpdateProposal.proposer = msg.sender;
        usdUpdateProposal.proposedPrice = _price;
        usdUpdateProposal.contracts = _contracts;
        usdUpdateProposal.approvals[msg.sender] = _usdProposalId;
        usdUpdateProposal.approvalsCount++;

        emit ProposedUSDPrice(msg.sender, _price);
    }
```

## Tools Used
Manual review.

## Recommended Mitigation Steps

Add appropriate checks in the `LockManager.sol::proposeUSDPrice` function to ensure that the price can never be set to 0.

The following check could be added before the event ```ProposedUSDPrice```:

```solidity
/// @inheritdoc ILockManager
    function proposeUSDPrice(
        uint256 _price,
        address[] calldata _contracts
    )
        external
        onlyOneOfRoles(
            [
                Role.PriceFeed_1,
                Role.PriceFeed_2,
                Role.PriceFeed_3,
                Role.PriceFeed_4,
                Role.PriceFeed_5
            ]
        )
    {
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
+       /// New condition here to check for price not equal 0
+       if (_price == 0) revert PriceCannotBeZeroError();
        if (_contracts.length == 0) revert ProposalInvalidContractsError();
        delete usdUpdateProposal;
                // Approvals will use this because when the struct is deleted the approvals remain
        ++_usdProposalId;

        usdUpdateProposal.proposedDate = uint32(block.timestamp);
        usdUpdateProposal.proposer = msg.sender;
        usdUpdateProposal.proposedPrice = _price;
        usdUpdateProposal.contracts = _contracts;
        usdUpdateProposal.approvals[msg.sender] = _usdProposalId;
        usdUpdateProposal.approvalsCount++;

        emit ProposedUSDPrice(msg.sender, _price);
    }
```

# LOW-2: Unchecked array length could breach gas block limit

Links to affected code:
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L142
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L177

## Impact

**Likelihood: LOW**
Arrays can only be set by one of the trusted roles.

**Impact: LOW**

**Gas Costs**: Each iteration over the ```_contracts``` array involves updating the USD price for a contract, which consumes gas. A very large array could lead to excessive gas costs, potentially making the transaction prohibitively expensive or even exceeding block gas limits, causing the transaction to fail.

## Proof of Concept
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L142

No checks are done in the ```LockManager.sol::proposeUSDPrice``` function for the array size in ```address[] calldata _contracts```.

```solidity
/// @inheritdoc ILockManager
    function proposeUSDPrice(
        uint256 _price,
        address[] calldata _contracts
    )
        external
        onlyOneOfRoles(
            [
                Role.PriceFeed_1,
                Role.PriceFeed_2,
                Role.PriceFeed_3,
                Role.PriceFeed_4,
                Role.PriceFeed_5
            ]
        )
    {
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
        if (_contracts.length == 0) revert ProposalInvalidContractsError();
        delete usdUpdateProposal;

        // Approvals will use this because when the struct is deleted the approvals remain
        ++_usdProposalId;

        usdUpdateProposal.proposedDate = uint32(block.timestamp);
        usdUpdateProposal.proposer = msg.sender;
        usdUpdateProposal.proposedPrice = _price;
        usdUpdateProposal.contracts = _contracts;
        usdUpdateProposal.approvals[msg.sender] = _usdProposalId;
        usdUpdateProposal.approvalsCount++;

        emit ProposedUSDPrice(msg.sender, _price);
    }
```

## Tools Used
Manual review.

## Recommended Mitigation Steps

Add appropriate checks in the `LockManager.sol::proposeUSDPrice` function to ensure that the array cannot exceed a certain limit.

The following check could be added before the event ```ProposedUSDPrice```:

```solidity
/// @inheritdoc ILockManager
    function proposeUSDPrice(
        uint256 _price,
        address[] calldata _contracts
    )
        external
        onlyOneOfRoles(
            [
                Role.PriceFeed_1,
                Role.PriceFeed_2,
                Role.PriceFeed_3,
                Role.PriceFeed_4,
                Role.PriceFeed_5
            ]
        )
    {
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
+       /// New condition here to check for array size
+       if (_contracts.length >= MaxArraySizeLimit) revert ArrayToBigError();
        if (_contracts.length == 0) revert ProposalInvalidContractsError();
        delete usdUpdateProposal;
                // Approvals will use this because when the struct is deleted the approvals remain
        ++_usdProposalId;

        usdUpdateProposal.proposedDate = uint32(block.timestamp);
        usdUpdateProposal.proposer = msg.sender;
        usdUpdateProposal.proposedPrice = _price;
        usdUpdateProposal.contracts = _contracts;
        usdUpdateProposal.approvals[msg.sender] = _usdProposalId;
        usdUpdateProposal.approvalsCount++;

        emit ProposedUSDPrice(msg.sender, _price);
    }
```