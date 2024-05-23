Issue: Lack of Event Emissions for Critical Actions

## Impact
Failing to emit events for critical actions reduces transparency and makes it harder to audit contract interactions.

## Tools Used
Manual code review

## Recommended Mitigation Steps

Add Event Emission:

Emit an event in the _lock function for locking tokens.

Emit Events for Critical Actions: Ensure events are emitted for actions like locking tokens to provide a transparent log.

Design Meaningful Event Structures: Capture all necessary details, such as token contract address, token owner, recipient, quantity, and unlock time.