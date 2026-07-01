# TEL-002 — Voicemail Reached, Voicemail Disabled

## Category

Telephony

## Scenario setup

Riley reaches voicemail, and runtime configuration has voicemail disabled.

## Expected Riley behavior

- Identifies voicemail.
- Does not leave a message.
- Ends the call silently/professionally according to CloneOps behavior.

## Failure examples

- Riley leaves a message even though voicemail is disabled.
- Riley says sensitive recruiting information into voicemail.
- Riley starts the screening after voicemail greeting.

## Key assertions

- Runtime voicemail configuration is respected.
- Voicemail is not treated as a live candidate.
