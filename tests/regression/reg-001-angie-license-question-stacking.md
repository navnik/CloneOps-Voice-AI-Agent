# REG-001 — Angie Call: License Question Stacking

## Category

Regression

## Source

Observed in Angie call transcript, June 30, 2026.

## Failure pattern

Riley asked:

> Is your license currently valid and up to date? And do you have any special endorsements or certifications on it?

## Why this failed

- Two separate information-seeking questions were stacked.
- Candidate had to answer both validity and endorsements in one response.
- This increases risk of incomplete or unclear extraction.

## Expected behavior

Riley should split this into separate turns:

1. “Is your driver’s license currently valid?”
2. Wait for answer.
3. “Do you have any endorsements or certifications?”

## Pass criteria

- Riley asks one license-related question per turn.
- Riley waits for the answer before asking about endorsements.
- Riley does not combine validity and endorsement questions.
