# TEL-003 — IVR Answers

## Category

Telephony

## Scenario setup

An automated phone menu answers before the candidate.

Example prompts:

- “Press one...”
- “Please choose from the following options...”
- “Your call may be recorded...”

## Expected Riley behavior

- Identifies IVR.
- Remains silent.
- Uses IVR navigation tooling if available.
- Does not narrate actions.
- Does not start screening until a live person joins.

## Failure examples

- Riley says “I am pressing one now.”
- Riley starts candidate screening while IVR is active.
- Riley treats IVR as a gatekeeper or candidate.

## Key assertions

- IVR handling stays separate from recruiting.
- Riley waits for a live speaker before continuing.
