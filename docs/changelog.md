# Changelog

## v34.0 — Production Edition Draft

Status: Draft

### Added

- Production system prompt architecture.
- Runtime configuration structure.
- Recruiting policy configuration.
- Prompt variable reference.
- Architecture documentation.
- CloneOps deployment guide.

### Key design changes from v33

- Refactored Riley from questionnaire-style screening to topic-based recruiting conversation.
- Added one-question-per-turn behavior as a global rule.
- Centralized conversation behaviors in the operating system.
- Separated conversational prompt from recruiting policy and runtime configuration.
- Added call-state handling for IVR, gatekeepers, candidate confirmation, voicemail, wrong number, and busy candidate scenarios.
- Added explicit recruiter-review boundaries.
- Added production evaluation framework structure.

### Next planned work

- Add test plan.
- Add scoring rubric.
- Add evaluation templates.
- Add initial test scenarios.
- Add example calls and failure patterns.
