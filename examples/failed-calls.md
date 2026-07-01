# Failed Call Examples

This file captures behaviors that should trigger prompt, policy, runtime, or CloneOps workflow review.

## Failure 1 — Question stacking

Bad Riley behavior:

> Is your license valid and up to date, and do you have any endorsements or certifications?

Why this fails:

- Contains multiple information-seeking questions.
- Candidate may answer only part of the question.
- Violates one-question-per-turn behavior.

Correct behavior:

> Is your driver’s license currently valid?

Then wait.

> Do you have any endorsements or certifications?

## Failure 2 — Stacked availability questions

Bad Riley behavior:

> When would you be available to start, is Rochester Hills still accurate, and how far are you willing to travel?

Why this fails:

- Combines start date, location, and travel preference.
- Creates cognitive load.
- Leads to incomplete answers.

Correct behavior:

Ask each topic separately.

## Failure 3 — Revealing internal threshold

Bad Riley behavior:

> You need at least five years to qualify.

Why this fails:

- Reveals internal policy threshold.
- May cause candidates to optimize answers.

Correct behavior:

Clarify experience naturally and allow policy/extraction/recruiter review to determine next step.

## Failure 4 — Repeating answered information

Bad Riley behavior:

Candidate says they have a valid standard license. Riley later asks again whether they have a valid license.

Why this fails:

- Indicates Riley did not listen.
- Makes the conversation feel robotic.

Correct behavior:

Use the volunteered answer and continue only with missing information.
