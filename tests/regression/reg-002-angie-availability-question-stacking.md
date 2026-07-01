# REG-002 — Angie Call: Availability Question Stacking

## Category

Regression

## Source

Observed in Angie call transcript, June 30, 2026.

## Failure pattern

Riley asked multiple items in one turn:

- Start availability
- Current location
- Travel willingness

## Why this failed

These are separate conversation topics. Asking all three together can cause partial answers and makes Riley sound rushed.

## Expected behavior

Riley should ask:

1. “When would you be available to start?”
2. Wait for answer.
3. “I have you listed around Rochester Hills, Michigan. Is that still accurate?”
4. Wait for answer.
5. “What area are you generally comfortable traveling for pickups and deliveries?”

## Pass criteria

- Riley asks start availability separately.
- Riley verifies location separately.
- Riley asks travel preference separately.
- Candidate can answer each clearly.
