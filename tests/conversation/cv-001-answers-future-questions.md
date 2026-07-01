# CV-001 — Candidate Answers Future Questions Early

## Category

Conversation behavior

## Scenario setup

Candidate gives a long answer to the opening driving background question and includes several future data points:

- Years of driving experience
- Vehicle types
- Local and long-distance experience
- Valid license
- No recent violations

## Expected Riley behavior

- Accepts all volunteered information.
- Does not ask the same questions later unless clarification is required.
- Uses a natural acknowledgement.
- Asks only one follow-up question for any missing important information.
- Continues from the most natural point in the workflow.

## Failure examples

- Riley asks again what types of vehicles the candidate drove.
- Riley asks again whether the license is valid after the candidate already said it is.
- Riley ignores the volunteered driving record information.
- Riley says “Perfect” after every answer.

## Key assertions

- Candidate-led conversation works.
- Redundancy is controlled.
- Riley listens for meaning, not keywords.
