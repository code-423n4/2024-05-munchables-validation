# QA Report for Munchables

## QA-01 Missing `RemovedUSDProposal()` event in `execUSDPriceUpdate` and `proposeUSDPrice`

### Impact

`execUSDPriceUpdate` and `proposeUSDPrice` does not emit `RemovedUSDProposal()` event upon deleting
the `usdUpdateProposal`, misleading contracts that are listening to
this event.

### Proof of Concept

The `execUSDPriceUpdate` and `proposeUSDPrice` function delete the `usdUpdateProposal` in certain scenarios, however
they are not emitting the `RemovedUSDProposal()` event.

```solidity
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
```solidity
    function _execUSDPriceUpdate() internal {
        if (
            usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD &&
            usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD
        ) {
            uint256 updateTokensLength = usdUpdateProposal.contracts.length;
            for (uint256 i; i < updateTokensLength; i++) {
                address tokenContract = usdUpdateProposal.contracts[i];
                if (configuredTokens[tokenContract].nftCost != 0) {
                    configuredTokens[tokenContract].usdPrice = usdUpdateProposal
                        .proposedPrice;

                    emit USDPriceUpdated(
                        tokenContract,
                        usdUpdateProposal.proposedPrice
                    );
                }
            }

            delete usdUpdateProposal;
        }
    }
```
### Recommended Mitigation Steps

Emit `RemovedUSDProposal()` event in the functions in the following cases.

```diff
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
+       emit RemovedUSDProposal();

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
```diff
    function _execUSDPriceUpdate() internal {
    if (
        usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD &&
        usdUpdateProposal.disapprovalsCount < DISAPPROVE_THRESHOLD
    ) {
        uint256 updateTokensLength = usdUpdateProposal.contracts.length;
        for (uint256 i; i < updateTokensLength; i++) {
            address tokenContract = usdUpdateProposal.contracts[i];
            if (configuredTokens[tokenContract].nftCost != 0) {
                configuredTokens[tokenContract].usdPrice = usdUpdateProposal
                    .proposedPrice;

                emit USDPriceUpdated(
                    tokenContract,
                    usdUpdateProposal.proposedPrice
                );
            }
        }

        delete usdUpdateProposal; 
+       emit RemovedUSDProposal();
    }
}
```
