### Vulnerability: Does not check for duplicates

**Description:**

The vulnerability arises from the absence of a check for duplicate token contracts when configuring new tokens. Specifically, in the `configureToken` function, the contract does not verify if the `_tokenContract` being added already exists in the `configuredTokenContracts` array. This can lead to multiple entries of the same token contract in the array, causing potential issues with enumeration and token management.

**Impact:**

1. **Enumeration Issues:** Duplicate entries can cause problems when iterating over the `configuredTokenContracts` array, leading to redundant operations and potential logic errors.
2. **Increased Gas Costs:** Redundant entries can increase the gas costs for operations that involve iterating over the `configuredTokenContracts` array.
3. **Data Integrity:** The presence of duplicate entries can compromise the integrity of the data, leading to unexpected behavior in the contract's functionality.

**Recommendation:**

Implement a check to ensure that the `_tokenContract` being added does not already exist in the `configuredTokenContracts` array before pushing it. This can be done by iterating through the array and verifying the absence of the `_tokenContract` before adding it.