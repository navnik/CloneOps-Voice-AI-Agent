# Riley v34 Architecture

## Overview

Riley v34 is designed as a layered Voice AI recruiting system, not a monolithic prompt.

The system separates:

1. Conversation behavior
2. Recruiting workflow
3. Business policy
4. Runtime configuration
5. Evaluation and testing

This separation makes Riley easier to maintain, test, and evolve.

## Architecture layers

### 1. Stable conversational brain

Location: `system-prompt/riley-v34-production.md`

Owns:

- Riley's identity
- Conversation principles
- One-question-per-turn behavior
- Topic management
- Call handling behavior
- Recovery behavior
- Neutrality and honesty

This layer should change rarely.

### 2. Recruiting policy

Location: `policies/recruiting-policy.yaml`

Owns:

- Qualification thresholds
- Hard disqualifiers
- Review conditions
- Candidate-facing policy boundaries

This layer is business-owned and should be updated when PARS recruiting policy changes.

### 3. Runtime configuration

Location: `runtime/`

Owns:

- Feature flags
- Voicemail behavior
- Callback behavior
- Approved message snippets
- Candidate-specific runtime variables

This layer may change frequently.

### 4. Evaluation framework

Location: `tests/`

Owns:

- Test scenarios
- Scoring rubric
- Regression testing
- Conversation quality evaluation
- Recruiting decision evaluation

## Execution model

Every call follows this flow:

```text
Call starts
↓
Identify call state
↓
Reach and confirm candidate
↓
Run screening workflow by topic
↓
Apply decision rules
↓
Recover naturally when conversation changes
↓
Close with recruiter handoff
↓
Extract structured summary for recruiter review
```

## Core design principle

Riley should optimize for candidate understanding, not checklist completion.

Every question should improve Riley's understanding of the candidate. If it does not, Riley should not ask it.

## Maintainability rule

Each instruction should have one source of truth.

- Conversation behavior belongs in the system prompt.
- Business rules belong in policy files.
- Frequently changing copy and feature flags belong in runtime config.
- Quality expectations belong in tests.
