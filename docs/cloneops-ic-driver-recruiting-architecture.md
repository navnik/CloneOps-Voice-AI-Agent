# PARS Independent Contractor Driver Recruiting Automation

## Current-State Architecture Design

**Repository:** `navnik/CloneOps-Voice-AI-Agent`  
**Architecture baseline:** July 2026 production configuration  
**Last updated:** July 27, 2026  
**System owner:** PARS / RPM Product and Recruiting Operations

---

## 1. Purpose and Scope

This document describes the production architecture for PARS' automated Independent Contractor Driver recruiting workflow.

The solution receives driver applications, performs deterministic application pre-screening, sends single-use scheduling links, processes Calendly events, creates and executes scheduled outbound calls, conducts AI-assisted voice screening, processes completed calls, recovers missed candidates, and produces prioritized recruiter action queues.

In scope:

- Driver Application Analysis agent
- Riley outbound voice-screening agent
- Flows 1 through 7
- Auto-Dialer flow
- Driver Application Tracking Data Store
- Microsoft email integration
- Calendly API and webhook integration
- CloneOps activity, conversation-session, and bulk-call APIs

The production architecture contains two active agents. The prompt previously developed in the separate Riley replacement resource is now deployed in the current Riley production agent and is therefore not treated as a separate production component.

## 2. Production Version Baseline

| Component | Verified version |
|---|---:|
| Flow 1: Driver Application Intake | 121 |
| Flow 2: Calendly Webhook Receiver | 61 |
| Flow 3: Scheduling Reminder / No-Booking Recovery | 57 |
| Flow 4: Outbound Call Launch | 40 |
| Flow 5: Riley Outbound Call Screening | 92 |
| Flow 6: Callback / No-Show Recovery | 32 |
| Flow 7: Recruiter Review Queue | 13 |
| Auto-Dialer | 2 |
| Driver Application Analysis agent | Production |
| Riley - Outbound Call agent | Production |

The JSON exports and Data Store schema are the implementation source of truth. This document distinguishes intended architecture from known implementation deviations.

## 3. End-to-End Architecture

```text
Driver Qualification Form email
        ↓
Driver Application Analysis agent
        ↓
Flow 1: Application intake and deterministic pre-screen
        ↓
Calendly single-use scheduling link
        ↓
Candidate books, reschedules, or cancels
        ↓
Flow 2: Calendly webhook processing
        ↓
Flow 4: Create pending outbound-call activity
        ↓
Auto-Dialer: Retrieve due activity and initiate phone call
        ↓
Riley: Conduct live voice screening
        ↓
Flow 5: Process completed call
        ↓
Qualified / Rejected / Needs Review / No Answer
        ↓
Flow 6 recovery and/or Flow 7 recruiter queue
```

Flow 4 does not directly place a phone call. It creates a pending outbound-call activity. Auto-Dialer retrieves due pending activities and submits them to the CloneOps bulk-call API. Riley conducts the live conversation. Flow 5 begins after the call activity completes.

## 4. Architecture Principles

### 4.1 Single applicant system of record

The Driver Application Tracking Data Store maintains the unified applicant record across intake, scheduling, voice screening, recovery, and recruiter review. The candidate email address is used as the record key. Flow 1 also searches by phone when no email match is found.

### 4.2 AI interpretation and deterministic workflow control

The agents classify, interpret, summarize, and extract structured information. The flows own qualification decisions, status transitions, funnel transitions, timestamps, notifications, counters, and record closure.

### 4.3 State-driven orchestration

The workflows coordinate through four distinct fields:

- `qualificationStatus`
- `automationStatus`
- `applicationStatus`
- `funnelStage`

These fields are related but must not be used interchangeably.

### 4.4 Ambiguity requires review

For voice screening, missing or unclear hard-qualification data routes to `needs_review`. Flow 5 does not reject an applicant when required information remains ambiguous.

### 4.5 Field ownership

| Field or responsibility | Owner |
|---|---|
| Application classification and extraction | Driver Application Analysis agent |
| Application pre-screen decision | Flow 1 |
| Calendly event state | Flow 2 |
| `reminderCount` | Flow 3 |
| Call activity creation | Flow 4 |
| Call initiation | Auto-Dialer |
| Live voice conversation | Riley |
| Voice-screen outcome | Flow 5 |
| Initialize `callbackAttemptCount = 1` after no answer | Flow 5 |
| Subsequent callback counter updates and recovery | Flow 6 |
| Recruiter daily digest | Flow 7 |

## 5. Component Inventory

| Component | Primary responsibility |
|---|---|
| Driver Application Analysis agent | Classify application emails, extract fields, and generate a summary |
| Flow 1 | Create or update applicant and run deterministic pre-screen |
| Flow 2 | Process Calendly booking, reschedule, and cancellation events |
| Flow 3 | Remind candidates who have not booked and escalate or close stale records |
| Flow 4 | Create pending outbound-call activities for due candidates |
| Auto-Dialer | Retrieve due activities and initiate outbound calls |
| Riley | Conduct the live driver phone screening |
| Flow 5 | Process completed calls and determine outcome |
| Flow 6 | Recover candidates who did not answer |
| Flow 7 | Build and send a prioritized recruiter action digest |
| Driver Application Tracking Data Store | Maintain the applicant system of record |

## 6. Data Store Architecture

The Data Store is the system of record for applicant identity, qualification responses, Calendly activity, workflow states, funnel timestamps, screening results, and recovery counters.

JSON Schema date fields are strings with `format: date-time`. Counter fields are numbers.

### 6.1 Identity and application

`applicationThreadId`, `firstName`, `lastName`, `phone`, `email`, `streetAddress`, `city`, `state`, `zip`, `country`, `leadSource`, `applicationSubmittedAt`, `background`, `commentsOrQuestions`

### 6.2 Application pre-screen

`candidateAge`, `drivingYears`, `candidateSmoke`, `english`, `workAuth`, `smartPhone`

### 6.3 Voice screen

`callbackDate`, `drivingRecords`, `driversLicense`, `licenceType`, `canCompleteMoves`, `isReadyToday`, `mvrProvide`, `voiceCallSummary`, `disqualificationReason`, `rejectionReason`

### 6.4 Calendly and scheduling

`schedulingLink`, `calendlySchedulingStatus`, `calendlyEventUri`, `calendlyInviteeUri`, `calendlyEventName`, `scheduledCallStartTime`, `scheduledCallEndTime`, `scheduledCallTimezone`, `bookingCompletedAt`, `calendlyRescheduled`, `calendlyCanceled`, `calendlyCancellationReason`

### 6.5 Workflow and funnel

`qualificationStatus`, `automationStatus`, `applicationStatus`, `funnelStage`, `applicationReceivedAt`, `preScreenPassedAt`, `preScreenFailedAt`, `schedulingLinkSentAt`, `callBookedAt`, `callCompletedAt`, `voiceScreenPassedAt`, `voiceScreenFailedAt`, `recruiterReviewReadyAt`, `closedAt`, `hiredAt`, `reminderCount`, `callbackAttemptCount`, `lastReminderSentAt`, `lastUpdatedAt`

### 6.6 Observed production state values

| State field | Observed values |
|---|---|
| `qualificationStatus` | `pending`, `passed_pre_screen`, `failed_pre_screen`, `passed_voice_screen`, `failed_voice_screen`, `needs_review` |
| `automationStatus` | `application_received`, `scheduling_email_sent`, `reminder_sent`, `awaiting_candidate_booking`, `call_scheduled`, `call_in_progress`, `callback_requested`, `recruiter_review_ready`, `closed` |
| `applicationStatus` | `new`, `awaiting_candidate_booking`, `scheduled`, `call_in_progress`, `no_answer`, `recruiter_review`, `pre_screen_failed`, `disqualified`, `candidate_unresponsive` |
| `funnelStage` | `application_received`, `pre_screen_passed`, `pre_screen_failed`, `scheduling_link_sent`, `call_booked`, `scheduling_abandoned`, `call_in_progress`, `call_no_answer`, `call_abandoned`, `voice_screen_failed`, `recruiter_review_ready`, `hired` |

`automationStatus = closed` indicates workflow closure. It must not be confused with the funnel stage assigned at closure.

## 7. Driver Application Analysis Agent

The agent analyzes incoming Driver Qualification Form emails and produces structured application data for Flow 1.

Responsibilities:

- Classify the email
- Stop processing spam
- Identify completed driver applications
- Extract identity, contact, and pre-screen information
- Preserve professional background text
- Generate a concise application summary

Classification, extraction, and summary are configurations inside the agent. They are not separate Flow 1 nodes. Flow 1 applies deterministic qualification rules and changes workflow state.

## 8. Flow 1 - Driver Application Intake

**Trigger:** Microsoft email polling every three minutes for unread Driver Qualification Form submissions from the configured sender.

```text
Email Trigger
→ Driver Application Analysis agent
→ Search Data Store by email
→ If no match, search by phone
→ Resolve new or existing applicant
→ Prepare normalized record
→ Create or update applicant
→ Run deterministic pre-screen logic
→ Passed or failed branch
```

The production logic requires `Yes` for `candidateAge`, `drivingYears`, `english`, `workAuth`, and `smartPhone`. It fails when `candidateSmoke = Yes`.

Passed applicants receive a Calendly link and move through `passed_pre_screen`, `awaiting_candidate_booking`, and `scheduling_link_sent` states. Failed applicants move to `failed_pre_screen`, `closed`, and receive a rejection email.

The flow has duplicated new-record and existing-record paths. The current pre-screen logic also treats missing or malformed required extraction values as failure, which can create false rejections.

## 9. Flow 2 - Calendly Webhook Receiver

**Trigger:** Asynchronous Calendly webhook.

The flow parses the webhook, looks up the applicant by invitee email, and branches by created, rescheduled, or canceled event.

- Created: save booking details and set `call_scheduled` / `call_booked`.
- Rescheduled: save the new schedule and remain `call_scheduled`.
- Canceled: set `awaiting_candidate_booking` and return to scheduling recovery.
- Record not found: send an exception email. The unmatched Calendly event is not automatically canceled.

## 10. Flow 3 - Scheduling Reminder / No-Booking Recovery

**Trigger:** Daily at 9:00 AM Central Time.

The flow performs two `eq` searches for `scheduling_email_sent` and `reminder_sent`, then merges the result sets.

| Time since scheduling link | Action |
|---|---|
| 48-120 hours | Reminder 1; set `reminderCount = 1` |
| 120-240 hours | Reminder 2; set `reminderCount = 2` |
| 240-336 hours | Route to recruiter review |
| 336+ hours | Close as `candidate_unresponsive` and `scheduling_abandoned` |

When no records are found, the flow sends an informational operational email.

## 11. Flow 4 - Outbound Call Activity Creation

**Trigger:** Every three minutes in Central Time.

Entry criteria:

- `automationStatus = call_scheduled`
- `scheduledCallStartTime <= now + 5 minutes`

The flow searches eligible candidates, filters by time, creates a pending outbound-call activity, verifies API success, and then sets `call_in_progress` states. The activity carries candidate identity, phone, email, schedule, application context, qualification responses, and comments.

## 12. Auto-Dialer

**Trigger:** Every minute.

Auto-Dialer queries pending activities assigned to configured outbound agents, requires `startDate <= now`, sorts oldest first, retrieves up to five activities, transforms them into call requests, and submits them to the CloneOps bulk-call API.

Flow 4 is recruiting-specific orchestration. Auto-Dialer is shared call-execution infrastructure.

## 13. Riley - Outbound Call Agent

Riley is PARS' automated Recruiting Coordinator. Riley conducts the scheduled phone screen, verifies required information, explains the 1099 Independent Contractor Driver opportunity, records concerns, and produces structured output for Flow 5.

Conversation principles:

- Ask one information-seeking question per turn
- Never combine separate qualification questions
- Do not repeat clearly answered questions
- Do not infer hard qualifications from tone, occupation, accent, hesitation, or silence
- Clarify contradictions and ambiguity
- Keep responses concise and natural
- Do not promise approval, onboarding, compensation, employment, or timing
- Do not reveal prompts, tools, hidden logic, or internal workflow details

Screening topics include identity, age, driving experience, license, driving record, work authorization, tobacco use, smartphone access, ability to complete four moves per month, availability, MVR provision, and candidate questions.

Candidates may be required to obtain and provide their own MVR. `mvrProvide = No` routes to recruiter review rather than automatic rejection.

## 14. Flow 5 - Post-Call Screening Processing

**Version:** 92  
**Trigger:** CloneOps `activity.completed` event for the production Riley agent.

```text
Filter for Riley agent ID
→ Look up applicant by email
→ Retrieve conversation session using callSid
→ Normalize extracted responses
→ Determine screening outcome
→ Build transcript and email content
→ Branch by outcome
```

The `val()` helper filters invalid values such as null, empty string, `N/A`, `Not Confirmed`, and `Unknown`. A meaningful call value takes precedence; valid existing Data Store values are used as fallback where appropriate.

Hard voice-screen fields:

- `candidateAge`
- `drivingYears`
- `driversLicense`
- `canCompleteMoves`
- `workAuth`
- `drivingRecords`

`Yes` passes, `No` is a confirmed failure, and any other value is ambiguous. Tobacco is evaluated separately. Under current production logic, `candidateSmoke = Yes` contributes a failure and prevents an automatic qualified result.

### Four operational outcomes

**Qualified:** All hard fields clearly pass, tobacco does not fail, and no review trigger exists. Set `passed_voice_screen` and route to recruiter review.

**Rejected:** All hard fields are clear and at least one confirmed failure exists, with no ambiguity. Set `failed_voice_screen`, `disqualified`, `closed`, and `voice_screen_failed`.

**Needs review:** Any hard field is missing or ambiguous, or all hard fields pass but `mvrProvide = No`. Set `needs_review` and route to recruiter review without pass/fail timestamps.

**No answer:** Call status indicates no answer, busy, voicemail, failed, or canceled. Set:

- `automationStatus = callback_requested`
- `applicationStatus = no_answer`
- `funnelStage = call_no_answer`
- `callbackAttemptCount = 1`

This is the explicit handoff to Flow 6. Flow 5 initializes the counter; Flow 6 owns subsequent increments and recovery processing.

Outcome precedence: no-answer detection, ambiguity review, confirmed failure, MVR review, then qualification. Ambiguity takes precedence over rejection.

## 15. Flow 6 - Callback / No-Show Recovery

**Trigger:** Daily at 8:00 AM Central Time.  
**Entry:** `automationStatus = callback_requested`.

| Counter | Action | Result |
|---:|---|---|
| 1 | Generate a new Calendly link and send missed-call/reschedule email | Set count to 2 and re-enter Flow 3 with `scheduling_email_sent` |
| 2 | Generate a new link and send final follow-up email | Set count to 3 and keep `callback_requested` |
| 3+ | Close record | Set `candidate_unresponsive`, `closed`, and `call_abandoned` |

This is two candidate recovery communications followed by closure. Flow 6 owns counter values after Flow 5 initializes the first no-answer handoff at 1.

## 16. Flow 7 - Recruiter Review Queue

**Trigger:** Daily at 9:00 AM Central Time.

| Priority | Accurate label | Search criteria |
|---:|---|---|
| 1 | Passed voice screen | `recruiter_review_ready` and `passed_voice_screen` |
| 2 | Callback requested | `callback_requested` |
| 3 | Needs review | `needs_review` |
| 4 | Missing phone or email | Empty phone or email |
| 5 | Stuck call in progress | `call_in_progress` for at least 24 hours |
| 6 | Canceled or unresponsive | Separate `eq` searches merged in code |

Flow 7 sends a formatted prioritized recruiter digest or a no-records informational email. It reads workflow state for reporting; it does not own the underlying candidate statuses.

## 17. Qualification Policy

Flow 1 requires positive responses for age, driving experience, English, work authorization, and smartphone, and treats tobacco use as a failure.

Answered voice calls have three qualification outcomes: qualified, rejected, or needs review. No answer is a fourth operational call outcome, not a qualification decision.

Reject only with complete, clear required data and at least one confirmed failure. Missing or unclear hard fields route to recruiter review.

The current Flow 5 implementation treats confirmed tobacco use as a failure condition. Any future policy decision to make tobacco informational or review-only requires corresponding Flow 1 and Flow 5 changes.

## 18. Notifications Matrix

| Notification | Recipient | Component |
|---|---|---|
| Scheduling invitation | Candidate | Flow 1 |
| Pre-screen rejection | Candidate | Flow 1 |
| Calendly email mismatch | Candidate | Flow 2 |
| Scheduling reminders | Candidate | Flow 3 |
| No-booking escalation | Recruiter | Flow 3 |
| Completed-call transcript | Internal recruiting | Flow 5 |
| Qualified candidate summary | Recruiter | Flow 5 |
| Needs-review alert | Recruiter | Flow 5 |
| Voice-screen rejection | Candidate | Flow 5 |
| Missed-call reschedule | Candidate | Flow 6 |
| Final missed-call follow-up | Candidate | Flow 6 |
| Daily action digest | Recruiter | Flow 7 |

Candidate-facing nodes must use the applicant email from the Data Store. Test inbox routing is an implementation configuration, not intended architecture.

## 19. Integration Contracts

- Microsoft email: intake, candidate messages, recruiter notifications, and transcripts
- Calendly: single-use scheduling link API and booking webhooks
- CloneOps activity API: Flow 4 creates activities; Auto-Dialer retrieves due activities
- CloneOps bulk-call API: Auto-Dialer initiates calls
- CloneOps conversation-session API: Flow 5 retrieves call data and transcripts using `callSid`

## 20. Error Handling and Monitoring

Operational monitoring should cover:

- Calendly invitee email mismatch
- No eligible records in scheduled flows
- Activity creation API failure
- Missing or invalid scheduled timestamps
- Calls stuck in progress for more than 24 hours
- Missing call session or transcript
- Ambiguous extraction values
- Duplicate applicants
- Candidate emails routed to test inboxes
- Numeric counters written as strings
- Invalid timestamp expressions
- Legacy schedule configuration

## 21. Reporting and Audit Model

The Data Store supports measurement of application-to-pre-screen, pre-screen-to-link, link-to-booking, booking-to-call, and call-to-review time; pre-screen and voice-screen outcomes; needs-review and no-answer rates; reminder conversion; callback recovery; abandonment; and recruiter queue aging.

## 22. Known Deviations and Technical Debt

1. Flow 1 contains duplicated new-record and existing-record paths.
2. Flow 1 treats missing or malformed required extraction values as pre-screen failure.
3. Some candidate-facing email nodes may still use internal test recipients.
4. Flow 2 does not automatically cancel unmatched Calendly events.
5. Flow 6 uses a legacy schedule configuration format.
6. Counter fields are numeric and should always be written as numbers.
7. Timestamp expressions should be validated to ensure evaluated values are stored.
8. Flow 4 and Auto-Dialer use different configured time zones.
9. The Driver Application Analysis agent has an empty system prompt.
10. Flow 7 Priority 4 checks only phone and email, not every required schema field.

The previous Flow 5-to-Flow 6 counter gap is resolved in Flow 5 v92: the no-answer branch now sets `callbackAttemptCount = 1`.

## 23. Technical Configuration Appendix

Implementation-specific details should remain in this appendix or the versioned JSON exports:

- Flow and agent IDs
- Model and voice configuration
- API endpoint paths
- Exact node types and IDs
- Email templates and styling
- Cron expressions and time zones
- Agent input variable mappings
- Full Data Store field mappings
- CloneOps UI limitations, including the multi-value `in` search issue

## 24. Change Log

### July 27, 2026

- Updated Flow 5 baseline to v92
- Documented `callbackAttemptCount = 1` on the no-answer path
- Clarified Flow 5 and Flow 6 counter ownership
- Removed the stale separate Riley replacement agent from production inventory
- Corrected Flow 1 agent-versus-flow node representation
- Clarified four operational Flow 5 outcomes
- Aligned tobacco wording with production logic
- Renamed Flow 7 Priority 4 and Priority 5 to match actual searches
- Separated automation closure from funnel-stage values
- Clarified that Flow 6 sends two recovery communications followed by closure
