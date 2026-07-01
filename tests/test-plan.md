# Riley v34 Evaluation Test Plan

## Purpose

The test suite validates Riley's production behavior before deployment and after every prompt or policy change.

The goal is not only to verify that Riley collects information. The goal is to verify that Riley conducts a natural, professional, recruiter-like conversation while collecting accurate information.

## Test categories

### 1. Happy path

Tests normal calls with cooperative candidates.

Examples:

- Qualified candidate with clear answers.
- Experienced driver with CDL.
- Candidate with personal driving background only.
- Candidate already familiar with 1099 work.

### 2. Conversation behavior

Tests Riley's conversational quality.

Examples:

- Candidate answers future questions early.
- Candidate interrupts Riley.
- Candidate gives long answers.
- Candidate gives very short answers.
- Candidate changes subject.
- Candidate asks if Riley is AI.

### 3. Recruiting logic

Tests qualification and recruiter-review behavior.

Examples:

- Under driving experience threshold.
- Unclear accident history.
- Work authorization issue.
- Smartphone unavailable.
- Application discrepancy.

### 4. Telephony

Tests call-state handling.

Examples:

- Gatekeeper answers.
- IVR answers.
- Voicemail.
- Wrong number.
- Candidate is busy.
- Call disconnects mid-screening.

### 5. Hard disqualifier

Tests early-conclusion behavior when a confirmed hard disqualifier applies.

Examples:

- Suspended license confirmed.
- Refuses background check.
- Refuses MVR review.
- No work authorization.

### 6. Regression

Tests previously observed failures.

Examples:

- Riley stacks two or three questions.
- Riley repeats a question already answered.
- Riley reveals internal thresholds.
- Riley overuses “perfect” or “awesome.”
- Riley jumps topics abruptly.

## Evaluation cadence

Run the full suite:

- Before deploying v34.
- After every system prompt change.
- After recruiting policy changes that affect call behavior.
- After CloneOps model, voice, transcription, or turn-detection changes.

Run targeted regression tests after small changes.

## Pass criteria

A scenario passes when Riley:

- Follows one-question-per-turn behavior.
- Does not ask redundant questions.
- Stays on topic until sufficient understanding is reached.
- Handles candidate behavior naturally.
- Applies recruiter-review and hard-disqualifier logic correctly.
- Produces accurate recruiter-ready output.
- Does not reveal internal policy thresholds.
- Maintains a professional candidate experience.

## Failure handling

For every failed scenario, record:

- Scenario ID.
- Transcript excerpt.
- Failure category.
- Expected behavior.
- Actual behavior.
- Root cause hypothesis.
- Recommended fix.
- Whether the fix requires prompt, policy, runtime, or CloneOps workflow changes.
