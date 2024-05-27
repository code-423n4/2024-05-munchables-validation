# [01] Add input validation for the `setUSDThresholds` function

The `setUSDTresholds` function does not perform input validation on `the _approve` and `_disapprove` parameters:

```javascript
    function setUSDThresholds(uint8 _approve, uint8 _disapprove) external onlyAdmin {
        if (usdUpdateProposal.proposer != address(0)) {
            revert ProposalInProgressError();
        }
        APPROVE_THRESHOLD = _approve; 
        DISAPPROVE_THRESHOLD = _disapprove;

        emit USDThresholdUpdated(_approve, _disapprove);
    }
```

If, by mistake, `setUSDThresholds` is called with `_approve > 5` and `_disapprove > 5`, and one of the PriceFeeds submits a proposal, USD prices will not be possible to update anymore, since thresholds larger than 5 cannot be reached if only 5 PriceFeeeds exist that can vote.

To mitigate the issue, implement a check as follows:

```diff
    function setUSDThresholds(uint8 _approve, uint8 _disapprove) external onlyAdmin {
        if (usdUpdateProposal.proposer != address(0)) {
            revert ProposalInProgressError();
        }
+       if(_approve + _disapprove > 5) { revert; }
        APPROVE_THRESHOLD = _approve; 
        DISAPPROVE_THRESHOLD = _disapprove;

        emit USDThresholdUpdated(_approve, _disapprove);
    }
```


# [02] Rogue PriceFeeds can keep proposing unacceptable USD prices

Rogue PriceFeed can continuously propose new (unacceptable) USD prices by repeatedly calling `proposeUSDPrice`. By keeping the proposal process in an infinite loop, it can prevent legitimate proposals from being made.


```javascript
    function proposeUSDPrice(uint256 _price, address[] calldata _contracts)
        external
        onlyOneOfRoles([Role.PriceFeed_1, Role.PriceFeed_2, Role.PriceFeed_3, Role.PriceFeed_4, Role.PriceFeed_5])
    {
        if (usdUpdateProposal.proposer != address(0)) {
            revert ProposalInProgressError();
        }
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

To mitigate the issue, implement a cooldown period or limit the frequency of proposals by the same address.


[03] `proposeUSDPrice` should emit the token address(es) the proposal applies to

The `proposeUSDPrice` function enables PriceFeeds to propose new USD prices, and as input parameters, accepts the proposed price `_price` and `_contracts`, an array of tokens the proposal applies to.

However, upon a successful call to this function, only the address of the proposer and the proposed price are emitted:

```javascript
        emit ProposedUSDPrice(msg.sender, _price); 
```

To mitigate the issue, add the `_contracts` array as the 3rd emit parameter.

