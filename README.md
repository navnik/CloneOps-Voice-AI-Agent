# CloneOps Voice AI Agent

This repository contains the production artifacts for PARS' CloneOps Voice AI recruiting agent, Riley.

Riley is designed as a conversational screening agent for 1099 Independent Contractor Driver candidates. The project separates Riley into maintainable layers so conversation behavior, recruiting policy, and runtime configuration can evolve independently.

## Repository structure

```text
CloneOps-Voice-AI-Agent/
├── docs/
├── examples/
├── policies/
├── runtime/
├── system-prompt/
└── tests/
```

## Current release

- Agent: Riley
- Version: v34 Production Edition
- Platform: CloneOps
- Primary use case: PARS 1099 IC Driver phone screening

## Design principle

Riley should not behave like a questionnaire. Riley should behave like an experienced recruiting coordinator who understands the candidate, verifies important information, and hands accurate context to PARS recruiters.

## Artifact ownership

- `system-prompt/` contains the stable conversational brain.
- `policies/` contains business-owned recruiting rules.
- `runtime/` contains CloneOps variables, feature flags, and configurable wording.
- `tests/` contains the evaluation framework and scenario suite.
- `docs/` contains architecture and deployment guidance.
- `examples/` contains reference calls and failure patterns.

## Release approach

The current consolidated design document remains the design specification. The production system prompt is a refactored deployment artifact optimized for CloneOps and GPT-4.1 Realtime.
