# CloneOps Deployment Guide

## Purpose

This guide explains how to deploy Riley v34 in CloneOps using the production system prompt, recruiting policy, runtime variables, and evaluation framework.

## Deployment components

### 1. System prompt

Use:

`system-prompt/riley-v34-production.md`

Paste this into Riley's CloneOps system prompt field or equivalent agent instruction area.

### 2. Runtime variables

Map variables from:

`runtime/prompt-variables.md`

Recommended required variables:

- `firstName`
- `lastName`
- `candidateName`
- `phoneNumber`
- `applicationThreadId`
- `city`
- `state`
- `scheduledCallTime`

### 3. Runtime configuration

Use:

- `runtime/runtime-config.yaml`
- `runtime/feature-flags.yaml`

These values should be implemented as CloneOps configuration, data store fields, flow variables, or agent variables where supported.

### 4. Recruiting policy

Use:

`policies/recruiting-policy.yaml`

Keep policy thresholds outside the stable system prompt when possible.

## Recommended deployment workflow

1. Update Riley system prompt with `riley-v34-production.md`.
2. Confirm CloneOps variables are mapped correctly.
3. Confirm feature flags for IVR, voicemail, and callback behavior.
4. Run happy-path test scenarios.
5. Run conversation behavior tests.
6. Run telephony tests.
7. Run recruiter-review and hard-disqualifier tests.
8. Review structured extraction output.
9. Deploy to limited production.
10. Review first 10 real calls before broad rollout.

## Pre-production checklist

- [ ] Riley asks one question per turn.
- [ ] Riley does not stack questions.
- [ ] Riley confirms identity before screening.
- [ ] Riley does not reveal internal thresholds.
- [ ] Riley handles gatekeepers without screening them.
- [ ] Riley handles voicemail according to configuration.
- [ ] Riley defers pay, benefits, taxes, and contract details to recruiter.
- [ ] Riley creates recruiter-ready summaries.
- [ ] Riley flags unclear cases for recruiter review.
- [ ] Riley does not imply pass/fail.

## Rollback

If Riley v34 creates unexpected behavior, revert to the previous CloneOps prompt version and preserve call transcripts for analysis.

## Versioning

Use semantic-style prompt versions:

- `v34.0` — initial production edition.
- `v34.1` — bug fix / prompt correction.
- `v34.2` — minor behavior refinement.
- `v35.0` — major workflow or architecture change.
