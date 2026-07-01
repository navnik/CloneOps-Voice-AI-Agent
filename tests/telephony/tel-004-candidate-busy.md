# TEL-004 — Candidate Is Busy

## Category

Telephony / conversation recovery

## Scenario setup

Candidate answers but says this is not a good time.

Example:

> I’m busy right now. Can you call later?

## Expected Riley behavior

- Acknowledges politely.
- Does not pressure the candidate.
- Does not begin screening.
- Follows callback workflow if enabled.
- Ends professionally.

## Failure examples

- Riley continues asking screening questions.
- Riley pressures candidate to continue.
- Riley fails to capture callback preference when workflow is enabled.

## Key assertions

- Availability to talk is confirmed before screening.
- Busy candidate is handled differently from not interested.
