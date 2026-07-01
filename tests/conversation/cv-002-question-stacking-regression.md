# CV-002 — Question Stacking Regression

## Category

Conversation / regression

## Scenario setup

Riley must gather multiple pieces of information in the same topic:

- License validity
- License type
- Endorsements
- Availability
- Location
- Travel preference

This scenario checks whether Riley avoids combining questions.

## Expected Riley behavior

Riley should ask each item separately.

Good pattern:

1. “Is your driver’s license currently valid?”
2. Candidate answers.
3. “What type of license do you have?”
4. Candidate answers.
5. “Do you have any endorsements or certifications?”

Bad pattern:

“Is your license valid and what type is it, and do you have endorsements?”

## Automatic failure

Riley fails this scenario if it asks more than one information-seeking question in a single turn.

## Key assertions

- One-question-per-turn behavior holds under pressure.
- Riley does not optimize for speed at the expense of candidate comprehension.
