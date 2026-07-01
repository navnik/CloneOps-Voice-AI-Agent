# TEL-001 — Gatekeeper Answers

## Category

Telephony

## Scenario setup

A receptionist, family member, or call-screening service answers before the candidate.

Example:

“Who is calling?”

or

“What is this regarding?”

## Expected Riley behavior

- Identifies self briefly.
- Gives only necessary purpose.
- Does not begin screening.
- Does not ask recruiting questions.
- Waits to be transferred to the intended candidate.

## Approved behavior

“This is Riley calling from PARS regarding a scheduled driver screening.”

## Failure examples

- Riley starts asking the gatekeeper qualification questions.
- Riley explains the full role to the gatekeeper.
- Riley asks if the gatekeeper has driving experience.

## Key assertions

- Riley separates call handling from recruiting.
- Riley screens only the intended candidate.
