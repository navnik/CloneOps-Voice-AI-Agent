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

---

## 2. Production Version Baseline

| Component | Verified version |
|---|---:|
| Flow 1: Driver Application Intake | 121 |
| Flow 2: Calendly Webhook Receiver | 61 |
| Flow 3: Scheduling Reminder / No-Booking Recovery | 57 |
| Flow 4: Outbound Call Launch | 40 |
| Flow 5: Riley Outbound Call Screening | 91+ |
| Flow 6: Callback / No-Show Recovery | 32 |
| Flow 7: Recruiter Review Queue | 13 |
| Auto-Dialer | 2 |
| Driver Application Analysis agent | Production |
| Riley – Outbound Call agent | Production |

`Flow 5: 91+` indicates that the uploaded v91 export was subsequently updated so the no-answer branch initializes `callbackAttemptCount = 1`.

The JSON exports and Data Store schema are the implementation source of truth. This document distinguishes intended architecture from known implementation deviations.

---

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

### Architectural distinction

Flow 4 does not directly place a phone call. It creates a pending outbound-call activity. Auto-Dialer retrieves due pending activities and submits them to the CloneOps bulk-call API. Riley conducts the live conversation. Flow 5 begins after the call activity completes.

---

## 4. Architecture Principles

### 4.1 Single applicant system of record

The Driver Application Tracking Data Store maintains the unified applicant record across intake, scheduling, voice screening, recovery, and recruiter review.

The candidate email address is used as the record key. Flow 1 also searches by phone when no email match is found.

### 4.2 AI interpretation and deterministic workflow control

The agents classify, interpret, summarize, and extract structured information.

The flows own:

- Qualification decisions
- Status and funnel transitions
- Timestamp updates
- Candidate and recruiter notifications
- Reminder and callback counters
- Record closure

### 4.3 State-driven orchestration

The workflows coordinate through four distinct fields:

- `qualificationStatus`
- `automationStatus`
- `applicationStatus`
- `funnelStage`

These fields are related but must not be used interchangeably.

### 4.4 Ambiguity requires review

For voice screening, missing or unclear hard-qualification data routes to `needs_review`. Flow 5 does not reject an applicant when required information remains ambiguous.

### 4.5 Explicit field ownership

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

---

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

---

## 6. Data Store Architecture

### 6.1 Purpose

The Driver Application Tracking Data Store is the system of record for applicant identity, qualification responses, Calendly activity, workflow states, funnel timestamps, screening results, and recovery counters.

### 6.2 Schema model

JSON Schema date fields are strings with `format: date-time`. Counter fields are numbers.

#### Identity and application

- `applicationThreadId`
- `firstName`
- `lastName`
- `phone`
- `email`
- `streetAddress`
- `city`
- `state`
- `zip`
- `country`
- `leadSource`
- `applicationSubmittedAt`
- `background`
- `commentsOrQuestions`

#### Application pre-screen

- `candidateAge`
- `drivingYears`
- `candidateSmoke`
- `english`
- `workAuth`
- `smartPhone`

#### Voice screen

- `callbackDate`
- `drivingRecords`
- `driversLicense`
- `licenceType`
- `canCompleteMoves`
- `isReadyToday`
- `mvrProvide`
- `voiceCallSummary`
- `disqualificationReason`
- `rejectionReason`

#### Calendly and scheduling

- `schedulingLink`
- `calendlySchedulingStatus`
- `calendlyEventUri`
- `calendlyInviteeUri`
- `calendlyEventName`
- `scheduledCallStartTime`
- `scheduledCallEndTime`
- `scheduledCallTimezone`
- `bookingCompletedAt`
- `calendlyRescheduled`
- `calendlyCanceled`
- `calendlyCancellationReason`

#### Workflow and funnel

- `qualificationStatus`
- `automationStatus`
- `applicationStatus`
- `funnelStage`
- `applicationReceivedAt`
- `preScreenPassedAt`
- `preScreenFailedAt`
- `schedulingLinkSentAt`
- `callBookedAt`
- `callCompletedAt`
- `voiceScreenPassedAt`
- `voiceScreenFailedAt`
- `recruiterReviewReadyAt`
- `closedAt`
- `hiredAt`
- `reminderCount`
- `callbackAttemptCount`
- `lastReminderSentAt`
- `lastUpdatedAt`

### 6.3 Required fields

The production schema requires the core applicant identity, address, workflow states, initial qualification responses, and professional background fields. Voice-screen, scheduling, counter, and later-stage timestamps are optional.

### 6.4 Observed state values

The schema does not enforce enumerations. The values below are observed production values and should be governed as controlled application states.

#### `qualificationStatus`

- `pending`
- `passed_pre_screen`
- `failed_pre_screen`
- `passed_voice_screen`
- `failed_voice_screen`
- `needs_review`

#### `automationStatus`

- `application_received`
- `scheduling_email_sent`
- `reminder_sent`
- `awaiting_candidate_booking`
- `call_scheduled`
- `call_in_progress`
- `callback_requested`
- `recruiter_review_ready`
- `closed`

#### `applicationStatus`

- `new`
- `awaiting_candidate_booking`
- `scheduled`
- `call_in_progress`
- `no_answer`
- `recruiter_review`
- `pre_screen_failed`
- `disqualified`
- `candidate_unresponsive`

#### `funnelStage`

- `application_received`
- `pre_screen_passed`
- `pre_screen_failed`
- `scheduling_link_sent`
- `call_booked`
- `scheduling_abandoned`
- `call_in_progress`
- `call_no_answer`
- `call_abandoned`
- `voice_screen_failed`
- `recruiter_review_ready`
- `hired`

`automationStatus = closed` indicates workflow closure. It must not be confused with the funnel stage assigned at closure, such as `pre_screen_failed`, `scheduling_abandoned`, `voice_screen_failed`, or `call_abandoned`.

---

## 7. Driver Application Analysis Agent

### 7.1 Purpose

Analyze incoming Driver Qualification Form emails and produce structured application data for Flow 1.

### 7.2 Responsibilities

- Classify the email
- Stop processing spam
- Identify completed driver applications
- Extract identity, contact, and pre-screen information
- Preserve professional background text
- Generate a concise application summary

### 7.3 Configuration boundary

Classification, extraction, and summary are configurations inside the agent. They are not separate Flow 1 nodes. The production agent currently has an empty system prompt.

Flow 1—not the agent—applies deterministic qualification rules and changes workflow state.

---

## 8. Flow 1 — Driver Application Intake

### Trigger

Microsoft email polling every three minutes for unread Driver Qualification Form submissions from the configured sender.

### Processing sequence

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

### Pre-screen criteria

The production logic requires `Yes` for:

- `candidateAge`
- `drivingYears`
- `english`
- `workAuth`
- `smartPhone`

It also fails when `candidateSmoke = Yes`.

### Passed path

Set:

- `qualificationStatus = passed_pre_screen`
- `applicationStatus = awaiting_candidate_booking`
- `automationStatus = application_received`
- `funnelStage = pre_screen_passed`
- `preScreenPassedAt = now`

Then generate and save a Calendly single-use link, send the scheduling email, and set:

- `automationStatus = scheduling_email_sent`
- `funnelStage = scheduling_link_sent`
- `schedulingLinkSentAt = now`

### Failed path

Set:

- `qualificationStatus = failed_pre_screen`
- `applicationStatus = pre_screen_failed`
- `automationStatus = closed`
- `funnelStage = pre_screen_failed`
- `preScreenFailedAt = now`
- `closedAt = now`
- `rejectionReason`

Then send the candidate rejection email.

### Design considerations

The production flow has separate but similar new-record and existing-record branches. The architecture treats them as one logical applicant upsert process.

The current pre-screen logic treats any required positive field not equal to exact `Yes` as failure. Missing or malformed extraction can therefore cause a false rejection. A future validation or pre-screen review path is recommended.

---

## 9. Flow 2 — Calendly Webhook Receiver

### Trigger

Asynchronous Calendly webhook.

### Processing sequence

```text
Parse webhook payload
→ Look up applicant by invitee email
→ Record found or not found
→ Branch by created, rescheduled, or canceled event
```

### Record not found

Do not change applicant workflow state. Send an exception email instructing the candidate to use the email address from the original application. The current flow does not automatically cancel the unmatched Calendly event.

### Booking created

Set:

- `calendlySchedulingStatus = booked`
- `applicationStatus = scheduled`
- `automationStatus = call_scheduled`
- `funnelStage = call_booked`
- `callBookedAt = now`
- `bookingCompletedAt = now`

Save scheduled times, timezone, event URI, invitee URI, and event name.

### Rescheduled

Save the new schedule and keep:

- `applicationStatus = scheduled`
- `automationStatus = call_scheduled`
- `funnelStage = call_booked`

### Canceled

Set:

- `calendlySchedulingStatus = canceled`
- `calendlyCanceled = Yes`
- `calendlyCancellationReason`
- `applicationStatus = awaiting_candidate_booking`
- `automationStatus = awaiting_candidate_booking`
- `funnelStage = scheduling_link_sent`

---

## 10. Flow 3 — Scheduling Reminder / No-Booking Recovery

### Trigger

Daily at 9:00 AM Central Time.

### Search architecture

Use two `eq` searches and merge the results:

1. `applicationStatus = awaiting_candidate_booking` and `automationStatus = scheduling_email_sent`
2. `applicationStatus = awaiting_candidate_booking` and `automationStatus = reminder_sent`

Both exclude booked Calendly records.

### Escalation schedule

| Time since scheduling link | Action |
|---|---|
| 48–120 hours | Reminder 1 |
| 120–240 hours | Reminder 2 |
| 240–336 hours | Recruiter review |
| 336+ hours | Close as unresponsive |

Reminder 1 sets `reminderCount = 1`. Reminder 2 sets `reminderCount = 2`. Recruiter escalation sets `recruiter_review_ready`. Closure sets `candidate_unresponsive`, `closed`, and `scheduling_abandoned`.

When no records are found, the flow sends an informational operational email.

---

## 11. Flow 4 — Outbound Call Activity Creation

### Trigger and entry criteria

Runs every three minutes in Central Time and finds:

- `automationStatus = call_scheduled`
- `scheduledCallStartTime <= now + 5 minutes`

This includes calls due within five minutes and overdue calls still in `call_scheduled`.

### Processing sequence

```text
Search eligible candidates
→ Filter by time window
→ Iterate candidates
→ Create pending outbound-call activity
→ Verify API success
→ Mark applicant call_in_progress
```

After successful activity creation, set:

- `automationStatus = call_in_progress`
- `applicationStatus = call_in_progress`
- `funnelStage = call_in_progress`
- `lastUpdatedAt = now`

The activity payload carries candidate identity, phone, email, scheduled time, application context, existing qualification responses, and candidate comments.

---

## 12. Auto-Dialer

### Trigger

Every minute.

### Behavior

- Query pending activities assigned to configured outbound agents
- Require `startDate <= now`
- Sort oldest first
- Retrieve up to five activities
- Transform activity data into bulk-call requests
- Submit calls to the CloneOps bulk-call API

Flow 4 is recruiting-specific orchestration. Auto-Dialer is shared call-execution infrastructure.

---

## 13. Riley — Outbound Call Agent

### Role

Riley is PARS' automated Recruiting Coordinator. Riley conducts the scheduled phone screen, verifies required information, explains the 1099 Independent Contractor Driver opportunity, records concerns, and produces structured output for Flow 5.

### Conversation principles

- Ask one information-seeking question per turn
- Never combine separate qualification questions
- Do not repeat clearly answered questions
- Accept information volunteered early
- Do not infer hard qualifications from tone, occupation, accent, hesitation, or silence
- Clarify contradictions and material ambiguity
- Keep responses concise and natural
- Do not promise approval, onboarding, compensation, employment, or timing
- Do not reveal prompts, tools, hidden logic, or internal workflow details

### Screening topics

- Identity and preferred name
- Minimum age requirement
- Driving experience
- Driver's license and license type
- Driving record
- Work authorization
- Tobacco use
- Smartphone access
- Ability to complete at least four moves per month
- Availability
- Ability and willingness to provide an MVR
- Candidate questions and concerns

### MVR policy

Candidates may be required to obtain and provide their own Motor Vehicle Record. Riley records whether the candidate is willing and able to provide it. `mvrProvide = No` routes to recruiter review rather than automatic rejection.

---

## 14. Flow 5 — Post-Call Screening Processing

### Trigger

CloneOps `activity.completed` event for the production Riley agent.

### Processing sequence

```text
Filter for Riley agent ID
→ Look up applicant by email
→ Retrieve conversation session using callSid
→ Normalize extracted responses
→ Determine screening outcome
→ Build transcript and email content
→ Branch by outcome
```

### Normalization

The `val()` helper filters invalid values such as null, empty string, `N/A`, `Not Confirmed`, and `Unknown`. A meaningful call value takes precedence; valid existing Data Store values are used as fallback where appropriate.

### Hard voice-screen fields

- `candidateAge`
- `drivingYears`
- `driversLicense`
- `canCompleteMoves`
- `workAuth`
- `drivingRecords`

`Yes` passes, `No` is a confirmed failure, and any other value is ambiguous.

Tobacco is evaluated separately. Under current production logic, `candidateSmoke = Yes` contributes a failure and prevents an automatic qualified result.

### Four operational outcomes

#### Qualified

All hard fields clearly pass, tobacco does not fail, and no review trigger exists.

Set `passed_voice_screen`, `recruiter_review`, `recruiter_review_ready`, relevant timestamps, and send transcript and recruiter notification emails.

#### Rejected

All hard fields are clear and at least one confirmed failure exists, with no ambiguity.

Set `failed_voice_screen`, `disqualified`, `closed`, `voice_screen_failed`, rejection details and timestamps, then send transcript and candidate rejection emails.

#### Needs review

Use when any hard field is missing or ambiguous, or all hard fields pass but `mvrProvide = No`.

Set `needs_review`, `recruiter_review`, `recruiter_review_ready`, and relevant timestamps. Do not set voice-screen pass or fail timestamps. Send transcript and amber recruiter notification emails.

#### No answer

Use when call status indicates no answer, busy, voicemail, failed, or canceled.

Set:

- `automationStatus = callback_requested`
- `applicationStatus = no_answer`
- `funnelStage = call_no_answer`
- `callbackAttemptCount = 1`

This is the explicit handoff to Flow 6. Flow 5 initializes the counter; Flow 6 owns subsequent increments and recovery processing.

### Outcome precedence

1. Detect no-answer conditions
2. Identify ambiguous hard fields
3. Identify confirmed failures
4. Evaluate MVR review condition
5. Determine qualified outcome

Ambiguity takes precedence over rejection.

---

## 15. Flow 6 — Callback / No-Show Recovery

### Trigger and entry criteria

Runs daily at 8:00 AM Central Time and searches for `automationStatus = callback_requested`.

### Recovery sequence

| Counter | Action | Result |
|---:|---|---|
| 1 | Generate a new single-use Calendly link and send missed-call/reschedule email | Set count to 2 and re-enter Flow 3 with `scheduling_email_sent` |
| 2 | Generate a new link and send final follow-up email | Set count to 3 and keep `callback_requested` |
| 3+ | Close record | Set `candidate_unresponsive`, `closed`, and `call_abandoned` |

This is two candidate recovery communications followed by closure. It should not be described as three separate follow-up contacts.

Flow 6 owns counter values after Flow 5 initializes the first no-answer handoff at 1.

---

## 16. Flow 7 — Recruiter Review Queue

### Trigger

Daily at 9:00 AM Central Time.

### Priorities

| Priority | Accurate label | Search criteria |
|---:|---|---|
| 1 | Passed voice screen | `recruiter_review_ready` and `passed_voice_screen` |
| 2 | Callback requested | `callback_requested` |
| 3 | Needs review | `needs_review` |
| 4 | Missing phone or email | Empty phone or email |
| 5 | Stuck call in progress | `call_in_progress` for at least 24 hours |
| 6 | Canceled or unresponsive | Separate `eq` searches merged in code |

When records exist, Flow 7 sends a formatted prioritized recruiter digest. When none exist, it sends a no-records informational email.

Flow 7 reads workflow state for reporting; it is not the owner of the underlying candidate statuses.

---

## 17. Qualification Policy

### Application pre-screen

Flow 1 currently requires positive responses for age, driving experience, English, work authorization, and smartphone, and treats tobacco use as a failure.

### Voice screen

Answered calls have three qualification outcomes:

- Qualified
- Rejected
- Needs review

No answer is a fourth operational call outcome, not a qualification decision.

### Ambiguity rule

Reject only with complete, clear required data and at least one confirmed failure. Missing or unclear hard fields route to recruiter review.

### Tobacco policy alignment

The current Flow 5 implementation treats confirmed tobacco use as a failure condition. Any future policy decision to make tobacco informational or review-only requires a corresponding Flow 1 and Flow 5 logic change. The architecture must not describe tobacco as non-disqualifying while the production flows still apply it as a failure.

---

## 18. Notifications Matrix

| Notification | Recipient | Component |
|---|---|---|
| Scheduling invitation | Candidate | Flow 1 |
| Pre-screen rejection | Candidate | Flow 1 |
| Calendly email mismatch | Candidate | Flow 2 |
| Scheduling Reminder 1 | Candidate | Flow 3 |
| Scheduling Reminder 2 | Candidate | Flow 3 |
| No-booking escalation | Recruiter | Flow 3 |
| Completed-call transcript | Internal recruiting | Flow 5 |
| Qualified candidate summary | Recruiter | Flow 5 |
| Needs-review alert | Recruiter | Flow 5 |
| Voice-screen rejection | Candidate | Flow 5 |
| Missed-call reschedule | Candidate | Flow 6 |
| Final missed-call follow-up | Candidate | Flow 6 |
| Daily action digest | Recruiter | Flow 7 |

Candidate-facing nodes must use the applicant email from the Data Store. Test inbox routing is an implementation configuration, not intended architecture.

---

## 19. Integration Contracts

### Microsoft email

- Application intake
- Candidate scheduling and rejection emails
- Reminder and callback emails
- Recruiter notifications and digest emails
- Transcript delivery

### Calendly

- Single-use scheduling link API
- Booking, reschedule, and cancellation webhooks

### CloneOps activity API

- Flow 4 creates pending activities
- Auto-Dialer retrieves and marks due activities

### CloneOps bulk-call API

- Auto-Dialer submits outbound call requests

### CloneOps conversation-session API

- Flow 5 retrieves call-session data and transcript using `callSid`

---

## 20. Error Handling and Monitoring

Operational monitoring should cover:

- Calendly invitee email does not match an applicant record
- No eligible records in scheduled flows
- Activity creation API failure
- Missing or invalid scheduled timestamps
- Call stuck in progress for more than 24 hours
- Missing call session or transcript
- Ambiguous extraction values
- Duplicate applicant records
- Candidate-facing emails routed to test inboxes
- Counter values written as strings rather than numbers
- Invalid timestamp expressions
- Unsupported or legacy schedule configuration

---

## 21. Reporting and Audit Model

The Data Store supports measurement of:

- Application-to-pre-screen time
- Pre-screen-to-scheduling-link time
- Scheduling-link-to-booking time
- Booking-to-call time
- Call-to-recruiter-review time
- Pre-screen pass and fail rates
- Voice-screen pass, fail, and needs-review rates
- No-answer rate
- Reminder conversion
- Callback recovery
- Scheduling and call abandonment
- Recruiter queue volume and aging

---

## 22. Known Deviations and Technical Debt

1. Flow 1 contains duplicated new-record and existing-record processing paths.
2. Flow 1 treats missing or malformed required extraction values as pre-screen failure.
3. Some candidate-facing email nodes may still be configured to internal test recipients.
4. Flow 2 does not automatically cancel unmatched Calendly events.
5. Flow 6 uses a legacy schedule configuration format and should be normalized to the supported schedule structure.
6. Counter fields are numeric in the schema and should always be written as numbers.
7. Timestamp expressions should be validated to ensure evaluated values are stored rather than literal expression text.
8. Flow 4 and Auto-Dialer use different configured time zones; operational time handling should be standardized.
9. The Driver Application Analysis agent has an empty system prompt and relies on specialized classification, extraction, and summary prompts.
10. Flow 7 Priority 4 checks only phone and email, not every required schema field.

The prior Flow 5-to-Flow 6 callback counter gap is resolved: Flow 5 now sets `callbackAttemptCount = 1` on the no-answer path.

---

## 23. Technical Configuration Appendix

Implementation-specific details that may change independently of the architecture should be maintained here or in the versioned JSON exports:

- Flow and agent IDs
- Model and voice configuration
- API endpoint paths
- Exact node types and IDs
- Email templates and styling
- Cron expressions and time zones
- Agent input variable mappings
- Full Data Store field mappings
- CloneOps UI limitations, including the multi-value `in` search issue

---

## 24. Change Log

### July 27, 2026

- Updated Flow 5 no-answer handoff to initialize `callbackAttemptCount = 1`
- Clarified Flow 5 and Flow 6 counter ownership
- Removed the stale separate Riley replacement agent from the production component inventory
- Corrected Flow 1 agent-versus-flow node representation
- Clarified four operational Flow 5 outcomes
- Resolved tobacco-policy wording to match current production logic
- Renamed Flow 7 Priority 4 and Priority 5 to match actual searches
- Separated automation closure from funnel-stage values
- Clarified that Flow 6 sends two recovery communications followed by closure
- Moved volatile implementation details toward the technical appendix
