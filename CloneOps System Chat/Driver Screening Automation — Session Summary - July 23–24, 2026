

---

# Driver Screening Automation — Session Summary (Continued)

## Date: July 23–24, 2026

## Operating Mode
**Consultative mode** — no changes to agents or flows unless explicitly asked. All flow changes require approval before proceeding. Never publish unless explicitly asked.

---

## What Was Reviewed This Session

| Resource | ID | Status |
|---|---|---|
| Riley — Outbound Call (current agent) | `6a29b3f1850b63147fbf8b03` | Reviewed, extraction fixes applied |
| New Agent (Riley's Replacement) | `6a624b7c9f3a8fad28629634` | Created, reviewed through multiple iterations, production-ready |
| Flow 5: Riley Outbound Call Screening | `6a38149fd29587a3b0f2b776` | **Modified** (v84 draft, production on v83) |
| Data Store: Driver Application Tracking | `6a2873ae79a7e88a96365765` | **Modified** — `mvrProvide` field added |

---

## Changes Made This Session

### Riley — Outbound Call (Extraction Config Fixes)
- **Added `candidateAge` and `drivingYears`** to the extraction prompt text — these fields were in the properties array but had no extraction instructions, causing 100% extraction failure across all 20 analyzed calls
- **Fixed `firstName` extraction prompt** — added handling for wrong-name edge case (Ashley/Minnie problem) and instructions to never leave blank when name was spoken
- **Fixed `candidateAge` property description** — changed from "21+" to "25+" to match system prompt
- **Fixed `drivingYears` property description** — changed from "minimum age requirement" (copy-paste error) to "at least five years of driving experience"
- **Fixed `canCompleteMoves` property description** — changed from "minimum age requirement" (copy-paste error) to "four moves per month" with implicit confirmation guidance
- **Added `firstName` property description** — was empty, now includes self-identified name preference

### New Agent (Riley's Replacement) — Multiple Iterations Reviewed
- System prompt reviewed through versions v1 → v35 → v36 → final production version
- Summary prompt completely rewritten
- Extraction prompt completely rewritten
- Properties array updated with cross-field inference guards and `mvrProvide`
- Final version is production-ready (see assessment below)

### Flow 5: Riley Outbound Call Screening (v84 draft)
- **Node 7 (Extract Structured Call Responses):** Added `mvrProvide: val(extracted.mvrProvide, null)` to return object
- **Update Record - Qualified:** Added `mvrProvide` to `data` object and `fieldMappings` array
- **Update Record - Rejected:** Added `mvrProvide` to `data` object and `fieldMappings` array

### Data Store: Driver Application Tracking
- **Added `mvrProvide` field** — STRING type, not required

---

## Extraction Analysis Findings (20 Calls Reviewed)

| Field | Total Calls | Always Empty | Reliable | Inconsistent | Root Cause |
|---|---|---|---|---|---|
| `candidateAge` | 20 | **20 (100%)** | 0 | 0 | Not in extraction prompt — **FIXED** |
| `drivingYears` | 20 | **20 (100%)** | 0 | 0 | Not in extraction prompt — **FIXED** |
| `candidateSmoke` | 20 | 0 | 17 | 3 | Riley sometimes skips the question |
| `firstName` | 20 | 4 | 15 | 1 wrong | Extraction missed confirmed names; one wrong-name bug — **FIXED** |
| `canCompleteMoves` | 20 | 0 | 15 | 3 | Bad property description + implicit confirmations missed — **FIXED** |
| `drivingRecords` | 20 | 0 | 18 | 2 | Vague answers → "Not Confirmed" (arguably correct) |
| `workAuth` | 20 | 0 | 19 | 1 | Riley skipped the question |

---

## New Agent (Riley's Replacement) — Final Assessment

### Key Improvements Over Current Riley

| Aspect | Current Riley | New Agent | Winner |
|---|---|---|---|
| Architecture | Flat checklist (Steps 1-8) | Goal-oriented sections with required outcomes | **New** |
| Priority hierarchy | None | Explicit 6-level priority stack | **New** |
| Age + work auth | Combined question (caused Jesus/Tom errors) | Separate questions, never combined | **New** |
| Tobacco coverage | In Step 2, sometimes skipped | Dedicated Section 8.4 with `{{usesTobacco}}` cross-check | **New** |
| Hard disqualifier criteria | Detailed list | Detailed list + structured 5-step follow-up process | **New** |
| Application cross-checking | `{{drivingExperienceQualified}}` and `{{usesTobacco}}` inline | All variables wired into workflow sections (8.2, 8.3, 8.4) | **New** |
| MVR handling | "PARS requires MVR review" | Candidate-provided MVR, legally precise language, approved wording | **New** |
| Anti-discrimination guardrails | None | Explicit in Sections 8.3, 8.7 | **New** |
| Prompt injection protection | None | Section 4 | **New** |
| Confidentiality | Detailed forbidden phrases | Detailed + downstream workflow protection | **New** |
| State integrity | Detailed | Detailed + early-termination handling | **New** |
| CDL conditional logic | Explicit if/then | Explicit in Section 8.5 | **Tied** |
| Completion standard | None | Section 12 — contract with extraction config | **New** |
| Summary prompt | 9 generic bullets | 17 specific items, MVR language controls, decision terminology | **New** |
| Extraction prompt | Basic field definitions | Per-field anti-inference guardrails, hard-disqualifier list mirrored, understanding vs. ability distinction | **New** |
| Cross-prompt alignment | Inconsistencies | Fully aligned across all three prompts | **New** |
| Prompt length / token cost | Shorter | Significantly longer | **Current** |

### New Extraction Fields
- **`mvrProvide`** — tracks whether candidate is willing and able to obtain and provide their own MVR. Full plumbing completed: extraction → properties array → data store → Flow 5.

### `canCompleteMoves` Behavior Change
The new extraction prompt distinguishes between understanding and ability:
- "Yes, I can do that" / "That sounds doable" → `Yes`
- "I understand" / "Okay" / "No questions" alone → `Not Confirmed` (unless surrounding context clearly communicates ability)
- This is stricter than the current prompt and may increase `needs_review` rate

---

## Decisions Made This Session

1. **Extraction fixes applied to current Riley** — `candidateAge`, `drivingYears`, `canCompleteMoves`, `firstName` descriptions and prompt text all corrected
2. **New agent reviewed through multiple iterations** — all previously flagged gaps addressed
3. **MVR changed from PARS-pulled to candidate-provided** — both current Riley (Step 7) and new agent (Section 8.8) now say MVR-only (no background check). New agent adds legally precise language about candidate obtaining their own MVR.
4. **`mvrProvide` field added end-to-end** — extraction → properties → data store → Flow 5 (qualified + rejected paths)
5. **Age and work auth must be separate questions** — new agent explicitly prohibits combining them
6. **Tobacco is not a hard disqualifier** — recorded for downstream review only, unless runtime policy overrides

---

## Unpublished Changes

| Resource | Draft Version | Production Version | Changes |
|---|---|---|---|
| Flow 5: Riley Outbound Call Screening | v84 | v83 | Added `mvrProvide` to extraction, qualified update, and rejected update nodes |
| Flow 3: Scheduling Reminder | v50 | v49 | From previous session — split `in` operator, added No Records email |
| Flow 6: Callback / No-Show Recovery | v32 | v31 | From previous session — added 3rd callback attempt |
| Flow 7: Recruiter Review Queue | v13 | v12 | From previous session — added P5 search, fixed P6, added No Records email |

---

## Unresolved Issues

### From This Session
1. **New agent not yet deployed to production** — needs test calls before swapping
2. **Flow 4 / Auto-Dialer agent ID** — when new agent replaces current Riley, Flow 4's agent filter and the Auto-Dialer need to be updated to point to the new agent ID (`6a624b7c9f3a8fad28629634`)
3. **Recruiter notification email doesn't include `mvrProvide`** — the qualified-path email template could be updated to show MVR willingness, but this was deferred

### Carried Forward from Previous Session
4. **Flow 1 & Flow 5 have active failures** — Flow 5 failed 5 times between July 6–15. Root cause not yet investigated.
5. **Flow 1: Duplicate logic paths** — New record and existing record paths have nearly identical node chains that can drift out of sync.
6. **Email agent has empty system prompt** — All intelligence is in the extraction/summary prompts.
7. **Flow 2: No Calendly event cancellation** when email mismatch is detected.
8. **Flow 3: Reminder emails go to test address** (`navnik.shivadas@parsinc.com`) instead of `{{item.data.email}}`.
9. **Flow 5: `callbackAttemptCount` hardcoded to `"1"` on no-answer** — should increment instead of reset.
10. **Flow 5: P3 (`needs_review`) will almost never surface** in Flow 7 because Flow 5's qualification logic overwrites Riley's `needs_review` status.
11. **Flow 4 + Auto-Dialer race condition** — Flow 4 marks `call_in_progress` after creating the activity, but Auto-Dialer might pick up the activity before the DS update completes.
12. **Flow 6 trigger config uses old cron format** (`"cron"` instead of `"cronExpressions"` array).
13. **Flow 2: `voiceCallSummary` cleared on every Calendly event** — would wipe existing summary if candidate reschedules after completing a call.
14. **Data store: `reminderCount` and `callbackAttemptCount` type mismatch** — schema says NUMBER but flows write strings.
15. **Data store: `preScreenPassedAt` stores literal `"$now()"` string** instead of actual timestamp.

---

## Next Steps (Suggested Priority)

1. **Run 5-10 test calls** with the new agent and compare extraction results against current Riley
2. **Publish Flow 5 (v84)** when ready — `mvrProvide` plumbing is complete
3. **Publish Flows 3, 6, 7** — still pending from previous session (v50, v32, v13)
4. **Swap production agent** — update Flow 4 / Auto-Dialer to point to new agent ID when test calls are satisfactory
5. **Fix Flow 3 reminder email recipients** — change `to` from test address to `{{item.data.email}}`
6. **Investigate Flow 5 failures** — check execution logs for the July 6–15 failures
7. **Fix `preScreenPassedAt` bug** in Flow 1
8. **Fix Flow 5 `callbackAttemptCount` increment**
9. **Fix Flow 6 trigger format**
10. **Add `mvrProvide` to recruiter notification email** (optional)

---

## Important Lessons Learned This Session

- **Extraction fields MUST be defined in both the prompt text AND the properties array** — having a field only in the properties array with a description is not sufficient; the LLM needs explicit extraction instructions in the prompt text
- **Copy-paste errors in property descriptions cause silent extraction failures** — the LLM sees conflicting signals (field name says one thing, description says another) and returns empty
- **Combined screening questions cause ambiguous answers** — the age + work auth combined question led to false disqualifications (Jesus said he was 40 but answered "No" to the combined question). Separate questions are more reliable.
- **"Understanding" ≠ "ability" for extraction** — candidates saying "I understand" or "okay" to a requirement doesn't necessarily confirm they can meet it. The new extraction prompt makes this distinction explicit.
- **MVR language matters legally** — "PARS requires an MVR review" implies PARS pulls it; "candidate may need to obtain and provide their own MVR" is the correct framing
- **Cross-prompt alignment is critical** — the system prompt, summary prompt, and extraction prompt must use consistent terminology, field names, and policy rules. Misalignment causes extraction drift.
- **New extraction fields need end-to-end plumbing** — adding a field to the extraction config is not enough; it must also be added to the properties array, data store schema, and any flows that read/write it
