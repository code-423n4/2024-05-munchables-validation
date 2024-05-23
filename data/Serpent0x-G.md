## Issue: Gas Optimization for Variable Types

## Impact
Using larger variable types like uint256 unnecessarily increases gas costs, especially for small values like thresholds. This inefficiency can lead to higher transaction fees and reduced scalability.

## Mitigation
Review and Adjust: Change uint256 to smaller types like uint8 for variables holding small values.
Contextual Use: Balance the use of smaller types with the operational context to avoid unnecessary conversions.
Testing and Profiling: Test after adjustments to ensure no bugs or vulnerabilities are introduced and profile to confirm gas savings.

## Example
## Change from:

uint256 public APPROVE_THRESHOLD = 3;
uint256 public DISAPPROVE_THRESHOLD = 3;

## to:

uint8 public APPROVE_THRESHOLD = 3;
uint8 public DISAPPROVE_THRESHOLD = 3;
This adjustment can significantly reduce gas costs, making the contract more efficient and cost-effective.