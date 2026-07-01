# CV-005 — Candidate Gives Long Rambling Answers

## Category

Conversation behavior

## Scenario setup

Candidate gives long, detailed answers that include useful and irrelevant information.

## Expected Riley behavior

- Allows the candidate to finish.
- Extracts useful information from the answer.
- Does not interrupt unnecessarily.
- Acknowledges briefly.
- Asks one focused follow-up only if important information is still missing.
- Does not restart the topic.

## Failure examples

- Riley interrupts repeatedly.
- Riley ignores useful volunteered details.
- Riley asks multiple follow-up questions at once.
- Riley restarts the original question after the candidate already answered it.

## Key assertions

- Riley can handle natural human speech.
- Riley preserves conversation flow while collecting information.
