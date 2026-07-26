# PARS Independent Contractor Driver Recruiting Automation

## Current-State Architecture Design

**Repository:** `navnik/CloneOps-Voice-AI-Agent`  
**Architecture baseline:** July 2026 production configuration  
**System owner:** PARS / RPM Product and Recruiting Operations

---

## 1. Purpose

This document describes the production architecture for PARS' automated Independent Contractor Driver recruiting workflow.

The solution receives driver applications, performs application pre-screening, sends single-use scheduling links, processes Calendly events, launches scheduled voice screenings, evaluates completed calls, recovers missed or unresponsive candidates, and creates prioritized recruiter action queues.

The architecture includes:

- Driver Application Analysis agent
- Riley outbound voice-screening agent
- Flows 1 through 7
- Auto-Dialer flow
- Driver Application Tracking Data Store
- Microsoft email integration
- Calendly API and webhook integration
- CloneOps activity, conversation, and calling APIs

---

## 2. Production Version Baseline

| Component | Production version |
|---|---:|
| Flow 1: Driver Application Intake | 121 |
| Flow 2: Calendly Webhook Receiver | 61 |
| Flow 3: Scheduling Reminder / No-Booking Recovery | 57 |
| Flow 4: Outbound Call Launch | 40 |
| Flow 5: Riley Outbound Call Screening | 91 |
| Flow 6: Callback / No-Show Recovery | 32 |
| Flow 7: Recruiter Review Queue | 13 |
| Auto-Dialer | 2 |
| Driver Application Analysis agent | Production |
| Riley – Outbound Call agent | Production |

The JSON exports and Data Store schema in the repository should be treated as the implementation source of truth. This document describes both the intended architecture and known production deviations.

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

### Important architectural distinction

Flow 4 does not directly place a phone call. It creates a pending outbound-call activity. The Auto-Dialer retrieves due activities and submits them to the CloneOps calling API. Riley conducts the live conversation. Flow 5 begins only after the activity completes.

---

## 4. Architecture Principles

### 4.1 Single applicant system of record

The Driver Application Tracking Data Store maintains the unified applicant record across application intake, scheduling, voice screening, recovery, and recruiter review.

The candidate email address is the primary record key. Flow 1 also searches by phone to detect an existing applicant when no email match is found.

### 4.2 AI interpretation and deterministic workflow control

The agents interpret unstructured communication and produce structured outputs.

The flows own:

- Qualification decisions
- Status transitions
- Funnel-stage transitions
- Timestamp updates
- Candidate communications
- Recruiter notifications
- Reminder and callback logic
- Record closure

The agents may apply only explicitly approved rules. They must not invent policy or make discretionary hiring decisions.

### 4.3 State-driven orchestration

The workflows coordinate primarily through four state fields:

- `qualificationStatus`
- `automationStatus`
- `applicationStatus`
- `funnelStage`

Each field has a distinct purpose and should not be used interchangeably.

### 4.4 Ambiguity requires human review

For voice screening, missing or ambiguous hard-qualification data must not be interpreted as a confirmed failure. Ambiguous calls route to `needs_review`.

### 4.5 Explicit field ownership

| Field or responsibility | Owner |
|---|---|
| Application extraction | Driver Application Analysis agent |
| Application pre-screen decision | Flow 1 |
| Calendly event state | Flow 2 |
| `reminderCount` | Flow 3 |
| Call activity creation | Flow 4 |
| Call initiation | Auto-Dialer |
| Live voice conversation | Riley |
| Voice-screen outcome | Flow 5 |
| `callbackAttemptCount` | Flow 6 |
| Recruiter daily digest | Flow 7 |

---

## 5. Component Inventory

| Component | Primary responsibility |
|---|---|
| Driver Application Analysis agent | Classify application emails, extract fields, and generate a summary |
| Flow 1 | Create or update applicant and run application pre-screen |
| Flow 2 | Process Calendly booking, reschedule, and cancellation events |
| Flow 3 | Remind candidates who have not booked and escalate or close stale records |
| Flow 4 | Create pending outbound-call activities for due candidates |
| Auto-Dialer | Retrieve due pending activities and initiate outbound calls |
| Riley | Conduct the live driver phone screening |
| Flow 5 | Process completed calls and determine workflow outcome |
| Flow 6 | Recover candidates who did not answer the scheduled call |
| Flow 7 | Build and send a prioritized recruiter action digest |
| Driver Application Tracking Data Store | Maintain the applicant system of record |

---

## 6. Data Store Architecture

### 6.1 Purpose

The Driver Application Tracking Data Store is the system of record for applicants throughout the automated recruiting process. It maintains identity, contact details, qualification responses, Calendly activity, workflow states, funnel timestamps, screening results, and recovery counters.

### 6.2 Identity and application fields

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

### 6.3 Application pre-screen fields

- `candidateAge`
- `drivingYears`
- `candidateSmoke`
- `english`
- `workAuth`
- `smartPhone`

### 6.4 Voice-screen fields

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

### 6.5 Calendly and scheduling fields

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

### 6.6 Workflow fields

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

### 6.7 Required fields

The production schema requires the core applicant identity, address, workflow states, initial qualification responses, and background fields. Optional fields are populated later by scheduling, voice screening, recovery, or recruiter activity.

### 6.8 Authoritative state values

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
- `closed`

---

## 7. Driver Application Analysis Agent

### 7.1 Purpose

The Driver Application Analysis agent analyzes incoming Driver Qualification Form emails and produces structured application data for Flow 1.

### 7.2 Responsibilities

- Classify the incoming email
- Identify spam and stop processing when appropriate
- Identify completed driver applications
- Extract candidate identity and contact information
- Extract initial qualification responses
- Preserve professional background text
- Generate a concise application summary

### 7.3 Current configuration model

The production agent stores most operational behavior in its classification, extraction, and summary configurations. Its system prompt is currently empty.

### 7.4 Architecture boundary

The agent extracts and summarizes information. Flow 1 applies the deterministic pre-screen criteria and performs all workflow actions.

---

## 8. Flow 1 — Driver Application Intake

### 8.1 Purpose

Receive application emails, create or update the applicant record, perform deterministic application pre-screening, and initiate scheduling for qualified applicants.

### 8.2 Trigger

Microsoft email polling every three minutes.

The trigger filters for:

- Unread email
- Driver Qualification Form subject
- Expected form-submission sender
- Recent processing window

### 8.3 Processing sequence

```text
Email Trigger
→ Driver Application Analysis agent
→ Search Data Store by email
→ If no match, search by phone
→ Resolve new or existing applicant
→ Prepare normalized applicant record
→ Create or update applicant
→ Run deterministic pre-screen logic
→ Passed or failed branch
```

### 8.4 Pre-screen criteria

The production logic requires `Yes` for:

- `candidateAge`
- `drivingYears`
- `english`
- `workAuth`
- `smartPhone`

The production logic also fails when:

- `candidateSmoke = Yes`

### 8.5 Passed path

Set:

- `qualificationStatus = passed_pre_screen`
- `applicationStatus = awaiting_candidate_booking`
- `automationStatus = application_received`
- `funnelStage = pre_screen_passed`
- `preScreenPassedAt = now`

Then:

1. Generate a Calendly single-use scheduling link.
2. Save the link.
3. Send the candidate a scheduling email.
4. Update:
   - `automationStatus = scheduling_email_sent`
   - `funnelStage = scheduling_link_sent`
   - `schedulingLinkSentAt = now`

### 8.6 Failed path

Set:

- `qualificationStatus = failed_pre_screen`
- `applicationStatus = pre_screen_failed`
- `automationStatus = closed`
- `funnelStage = pre_screen_failed`
- `preScreenFailedAt = now`
- `closedAt = now`
- `rejectionReason`

Then send the candidate a rejection email.

### 8.7 Known design consideration

The production flow has separate but similar new-record and existing-record branches. The architecture treats these as one logical applicant upsert process. Consolidating the duplicate paths would reduce configuration drift.

The current pre-screen code also treats any value other than exact `Yes` as failure. Extraction uncertainty should eventually be distinguished from an explicit negative response.

---

## 9. Flow 2 — Calendly Webhook Receiver

### 9.1 Purpose

Process Calendly booking, reschedule, and cancellation events and synchronize scheduling state with the applicant record.

### 9.2 Trigger

Asynchronous Calendly webhook.

### 9.3 Processing sequence

```text
Webhook Trigger
→ Parse Calendly payload
→ Look up applicant by invitee email
→ Record found or not found
→ Branch by event type
```

### 9.4 Record-not-found branch

If the invitee email does not match a Data Store record:

- Do not update applicant workflow state.
- Send an exception email instructing the candidate to use the email address from the original application.

The current production flow does not automatically cancel the unmatched Calendly event.

### 9.5 Booking-created branch

Set:

- `calendlySchedulingStatus = booked`
- `applicationStatus = scheduled`
- `automationStatus = call_scheduled`
- `funnelStage = call_booked`
- `callBookedAt = now`
- `bookingCompletedAt = now`

Save:

- Scheduled start and end
- Scheduled timezone
- Calendly event URI
- Calendly invitee URI
- Event name
- Cancel and reschedule context where available

### 9.6 Rescheduled branch

Update:

- New scheduled start and end
- Scheduled timezone
- Reschedule flags and related Calendly fields
- `applicationStatus = scheduled`
- `automationStatus = call_scheduled`
- `funnelStage = call_booked`

### 9.7 Canceled branch

Set:

- `calendlySchedulingStatus = canceled`
- `calendlyCanceled = Yes`
- `calendlyCancellationReason`
- `automationStatus = awaiting_candidate_booking`
- `applicationStatus = awaiting_candidate_booking`
- `funnelStage = scheduling_link_sent`

This returns the candidate to scheduling recovery.

---

## 10. Flow 3 — Scheduling Reminder / No-Booking Recovery

### 10.1 Purpose

Recover candidates who received a scheduling link but did not book a screening call.

### 10.2 Trigger

Daily at 9:00 AM Central Time.

### 10.3 Entry criteria

Search separately for applicants where:

- `applicationStatus = awaiting_candidate_booking`
- `automationStatus = scheduling_email_sent`
- Calendly is not booked

and:

- `applicationStatus = awaiting_candidate_booking`
- `automationStatus = reminder_sent`
- Calendly is not booked

The two result sets are merged before processing.

### 10.4 Escalation schedule

| Time since scheduling link | Action |
|---|---|
| 48–120 hours | Send Reminder 1 |
| 120–240 hours | Send Reminder 2 |
| 240–336 hours | Route to recruiter review |
| 336+ hours | Close as unresponsive |

### 10.5 Reminder 1

Set:

- `automationStatus = reminder_sent`
- `reminderCount = 1`
- `lastReminderSentAt = now`

### 10.6 Reminder 2

Set:

- `automationStatus = reminder_sent`
- `reminderCount = 2`
- `lastReminderSentAt = now`

### 10.7 Recruiter escalation

Set:

- `automationStatus = recruiter_review_ready`
- `applicationStatus = recruiter_review`
- `funnelStage = recruiter_review_ready`
- `recruiterReviewReadyAt = now`

### 10.8 Close as unresponsive

Set:

- `applicationStatus = candidate_unresponsive`
- `automationStatus = closed`
- `funnelStage = scheduling_abandoned`
- `closedAt = now`

### 10.9 Operational no-records branch

When no eligible applicants are found, the flow sends an informational operational email.

---

## 11. Flow 4 — Outbound Call Activity Creation

### 11.1 Purpose

Create a pending outbound-call activity for candidates whose scheduled screening time is due.

### 11.2 Trigger

Every three minutes in Central Time.

### 11.3 Entry criteria

- `automationStatus = call_scheduled`
- `scheduledCallStartTime <= now + 5 minutes`

The filter includes calls due within five minutes and overdue calls that remain in `call_scheduled`.

### 11.4 Processing sequence

```text
Scheduled Trigger
→ Search eligible candidates
→ Filter by call-time window
→ Check whether any are ready
→ Iterate candidates
→ Create pending outbound-call activity
→ Verify API result
→ Mark applicant call_in_progress
```

### 11.5 Activity payload

The activity carries the candidate context required by Riley, including:

- Candidate name
- Phone number
- Email
- Scheduled call time
- Existing qualification responses
- Applicant record context
- Calendly context
- Candidate notes or questions

### 11.6 Data Store update

After successful activity creation, set:

- `automationStatus = call_in_progress`
- `applicationStatus = call_in_progress`
- `funnelStage = call_in_progress`
- `lastUpdatedAt = now`

---

## 12. Auto-Dialer

### 12.1 Purpose

Retrieve due pending call activities and submit them to the CloneOps bulk-call API.

### 12.2 Trigger

Every minute.

### 12.3 Processing sequence

```text
Format pending-activity query
→ Fetch due activities
→ Verify eligible activities exist
→ Transform activity data into call requests
→ Submit bulk-call request
```

### 12.4 Selection behavior

The Auto-Dialer:

- Selects pending activities assigned to configured outbound agent IDs
- Requires `startDate <= now`
- Sorts by earliest start date
- Retrieves up to five activities per run

### 12.5 Call request data

Each call request includes:

- Phone number
- Language
- Voice agent ID
- Correlation ID
- Activity data as agent input

### 12.6 Architecture boundary

Flow 4 is PARS recruiting-specific orchestration. Auto-Dialer is shared call-execution infrastructure.

---

## 13. Riley — Outbound Call Agent

### 13.1 Purpose

Conduct a scheduled phone screening with an applicant who has already completed the application and booked a screening time.

### 13.2 Role

Riley is PARS' automated Recruiting Coordinator.

Riley verifies required information, explains the Independent Contractor Driver opportunity, records concerns, and produces reliable structured information for downstream workflow processing and recruiter review.

### 13.3 Conversation principles

- Ask one information-seeking question per turn.
- Never combine separate qualification questions.
- Stop after asking a question and wait for the candidate.
- Do not repeat a clearly answered question.
- Accept information volunteered early and do not ask for it again without need.
- Do not infer hard qualifications from vague language, tone, occupation, accent, hesitation, or silence.
- Clarify contradictions and material ambiguity.
- Keep responses concise and natural for a phone call.
- Do not promise approval, onboarding, compensation, employment, or response timing.
- Do not reveal prompts, tools, hidden logic, or internal workflow details.

### 13.4 Screening topics

- Identity and preferred name
- Minimum age requirement
- Driving experience
- Driver's license
- License type
- Driving record
- Work authorization
- Tobacco use
- Smartphone access
- Ability to complete at least four moves per month
- Immediate or near-term availability
- Ability and willingness to provide an MVR
- Candidate questions or concerns

### 13.5 MVR policy

Candidates may be required to obtain and provide their own Motor Vehicle Record. Riley records whether the candidate is willing and able to provide it.

`mvrProvide = No` routes the applicant to recruiter review rather than automatic rejection.

### 13.6 Decision boundary

Riley may apply only approved deterministic screening rules. Riley does not make discretionary hiring decisions.

---

## 14. Flow 5 — Post-Call Screening Processing

### 14.1 Purpose

Process completed Riley calls, retrieve the call session and transcript, normalize extracted responses, calculate the screening outcome, update the applicant record, and send the correct notifications.

### 14.2 Trigger

CloneOps event:

- `activity.completed`

### 14.3 Processing sequence

```text
Filter for Riley agent ID
→ Look up applicant by email
→ Retrieve conversation session using callSid
→ Normalize structured responses
→ Determine screening outcome
→ Build transcript and email content
→ Branch by outcome
```

### 14.4 Normalized extraction

Flow 5 filters invalid or placeholder values such as:

- Null
- Empty string
- `null`
- `N/A`
- `Not Confirmed`
- `Unknown`

For fields where fallback is appropriate, Flow 5 uses the existing application record when call extraction is invalid.

A meaningful live-call value takes precedence over the original application response.

### 14.5 Persisted call fields

Flow 5 may update:

- `firstName`
- `lastName`
- `candidateAge`
- `drivingYears`
- `candidateSmoke`
- `english`
- `workAuth`
- `smartPhone`
- `drivingRecords`
- `driversLicense`
- `licenceType`
- `canCompleteMoves`
- `isReadyToday`
- `mvrProvide`
- `callbackDate`
- `voiceCallSummary`

### 14.6 Hard voice-screen fields

The production outcome logic evaluates:

- `candidateAge`
- `drivingYears`
- `driversLicense`
- `canCompleteMoves`
- `workAuth`
- `drivingRecords`

Each required field is interpreted as:

- `Yes` — pass
- `No` — confirmed failure
- Any other value — ambiguous

Tobacco is evaluated separately. `candidateSmoke = Yes` contributes a failure under the current production logic.

### 14.7 Outcome precedence

1. Detect no-answer conditions.
2. Identify ambiguous required fields.
3. Identify explicit failures.
4. Evaluate MVR review condition.
5. Determine qualified outcome.

Ambiguity takes precedence over rejection. If a required field is ambiguous while another field explicitly fails, the applicant routes to `needs_review`.

### 14.8 Qualified path

Use when all hard fields clearly pass and no review trigger exists.

Set:

- `qualificationStatus = passed_voice_screen`
- `applicationStatus = recruiter_review`
- `automationStatus = recruiter_review_ready`
- `funnelStage = recruiter_review_ready`
- `callCompletedAt = now`
- `voiceScreenPassedAt = now`
- `recruiterReviewReadyAt = now`

Send:

- Transcript email
- Recruiter notification with screening responses and summary

### 14.9 Rejected path

Use only when all required hard fields are clear and at least one is an explicit failure.

Set:

- `qualificationStatus = failed_voice_screen`
- `applicationStatus = disqualified`
- `automationStatus = closed`
- `funnelStage = voice_screen_failed`
- `callCompletedAt = now`
- `voiceScreenFailedAt = now`
- `closedAt = now`
- `disqualificationReason`
- `rejectionReason`

Send:

- Transcript email
- Candidate rejection email

### 14.10 Needs-review path

Use when:

- Any required hard field is missing or ambiguous, or
- All hard fields pass but `mvrProvide = No`

Set:

- `qualificationStatus = needs_review`
- `applicationStatus = recruiter_review`
- `automationStatus = recruiter_review_ready`
- `funnelStage = recruiter_review_ready`
- `callCompletedAt = now`
- `recruiterReviewReadyAt = now`

Do not set:

- `voiceScreenPassedAt`
- `voiceScreenFailedAt`

Send:

- Transcript email
- Amber recruiter notification with review reasons

### 14.11 No-answer path

Use when the call result indicates no answer, busy, failed, canceled, or voicemail.

Set:

- `automationStatus = callback_requested`
- `applicationStatus = no_answer`
- `funnelStage = call_no_answer`

Flow 5 does not own `callbackAttemptCount`.

---

## 15. Flow 6 — Callback / No-Show Recovery

### 15.1 Purpose

Recover applicants who did not answer their scheduled voice-screening call.

### 15.2 Entry criteria

- `automationStatus = callback_requested`

### 15.3 Intended callback state machine

A blank or zero callback count should be treated as the first recovery attempt.

#### Recovery attempt 1

- Generate a new Calendly single-use link.
- Save the new scheduling link.
- Send a missed-call and rescheduling email.
- Re-enter the Flow 3 scheduling-reminder funnel.
- Increment `callbackAttemptCount`.

#### Recovery attempt 2

- Generate another single-use link.
- Send a final follow-up email.
- Increment `callbackAttemptCount`.

#### Close

After the configured maximum attempts:

- `applicationStatus = candidate_unresponsive`
- `automationStatus = closed`
- `funnelStage = call_abandoned`
- `closedAt = now`

### 15.4 Field ownership

Flow 6 is the sole owner of `callbackAttemptCount`.

---

## 16. Flow 7 — Recruiter Review Queue

### 16.1 Purpose

Create and send a daily prioritized action list for recruiting.

### 16.2 Trigger

Daily at 9:00 AM Central Time.

### 16.3 Priority groups

#### Priority 1 — Passed voice screen

- `automationStatus = recruiter_review_ready`
- `qualificationStatus = passed_voice_screen`

#### Priority 2 — Callback requested

- `automationStatus = callback_requested`

#### Priority 3 — Needs review

- `qualificationStatus = needs_review`

#### Priority 4 — Missing contact data

Production currently checks:

- Missing phone, or
- Missing email

#### Priority 5 — Stuck call

- `automationStatus = call_in_progress`
- `lastUpdatedAt` missing or at least 24 hours old

#### Priority 6 — Canceled or unresponsive

Includes applicants who are canceled or marked `candidate_unresponsive`.

### 16.4 Output

When records exist, send a formatted recruiter digest grouped by priority.

When no records exist, send an informational no-records email.

---

## 17. Qualification Policy

### 17.1 Application pre-screen

Flow 1 currently treats the following as required:

- Minimum age met
- Minimum driving experience met
- English fluency confirmed
- Work authorization confirmed
- Smartphone access confirmed
- No tobacco use

### 17.2 Voice screen

Flow 5 evaluates explicit call responses and differentiates:

- Confirmed pass
- Confirmed failure
- Missing or ambiguous information

### 17.3 Human-review boundary

Applicants route to recruiter review when:

- A required voice-screen field is ambiguous
- MVR provision is declined
- A workflow or data-quality exception requires human judgment

---

## 18. Notifications Matrix

| Notification | Recipient | Component |
|---|---|---|
| Scheduling invitation | Candidate | Flow 1 |
| Pre-screen rejection | Candidate | Flow 1 |
| Calendly email mismatch | Candidate | Flow 2 |
| Booking Reminder 1 | Candidate | Flow 3 |
| Booking Reminder 2 | Candidate | Flow 3 |
| No-booking escalation | Recruiter | Flow 3 |
| Qualified transcript | Internal recipient | Flow 5 |
| Qualified screening summary | Recruiter | Flow 5 |
| Needs-review transcript | Internal recipient | Flow 5 |
| Needs-review alert | Recruiter | Flow 5 |
| Voice-screen rejection | Candidate | Flow 5 |
| Missed-call reschedule | Candidate | Flow 6 |
| Daily recruiter digest | Recruiter | Flow 7 |

All candidate-facing production email nodes should use the candidate email from the Data Store. Test inbox addresses are non-production configuration and are not intended architecture.

---

## 19. Integration Contracts

### 19.1 Microsoft email

Used for:

- Application intake
- Candidate scheduling and rejection messages
- Reminder and callback messages
- Recruiter notifications
- Operational no-records notifications

### 19.2 Calendly

Used for:

- Single-use scheduling-link creation
- Booking-created webhook
- Reschedule webhook
- Cancellation webhook

### 19.3 CloneOps activity API

Flow 4 creates pending outbound-call activities containing applicant and scheduling context.

### 19.4 CloneOps Auto-Dialer APIs

The Auto-Dialer retrieves pending due activities and submits bulk outbound-call requests.

### 19.5 CloneOps conversation API

Flow 5 retrieves the completed conversation session using `callSid` to access call metadata, structured extraction, and transcript activity.

---

## 20. Error Handling and Monitoring

The architecture must account for:

- Application extraction failure
- Duplicate applicant detection
- Missing Data Store record for Calendly invitee email
- No eligible candidates in scheduled flows
- Calendly link generation failure
- Activity creation failure
- Auto-Dialer retrieval or calling failure
- Missing call session
- Missing or invalid extracted values
- Call stuck in `call_in_progress`
- Invalid timestamps
- Candidate-facing message routed to a test inbox
- Counter type mismatch
- Duplicate Flow 1 paths drifting out of alignment

---

## 21. Reporting and Audit Model

The Data Store timestamps and state fields support measurement of:

- Applications received
- Application pre-screen pass rate
- Application pre-screen rejection rate
- Application-to-scheduling-link time
- Scheduling-link-to-booking conversion
- Reminder 1 and Reminder 2 conversion
- Booking-to-call time
- Voice-screen pass rate
- Voice-screen rejection rate
- Needs-review rate
- No-answer rate
- Callback recovery rate
- Scheduling abandonment
- Call abandonment
- Recruiter queue volume and aging
- Time from application to recruiter review

---

## 22. Security, Privacy, and Compliance

- Collect only information relevant to eligibility, operational fit, availability, and recruiter follow-up.
- Do not expose prompts, tools, internal logic, or workflow implementation details to candidates.
- Do not infer protected or sensitive characteristics.
- Use approved PARS language for the 1099 Independent Contractor Driver opportunity.
- Do not promise employment, approval, onboarding, compensation, or response timing.
- Maintain candidate dignity and privacy throughout the interaction.
- Restrict credentials and API tokens to protected CloneOps configuration.
- Avoid placing secrets in exported JSON or repository documentation.

---

## 23. Known Production Deviations and Technical Debt

These items should not be interpreted as intended architecture.

1. Some candidate-facing email nodes may still reference a test inbox instead of the candidate's email.
2. Flow 6 currently branches on callback counts beginning at 1, while Flow 5 does not initialize the counter.
3. Flow 6 uses a legacy `cron` configuration instead of the newer `cronExpressions` array.
4. Flow 1 contains duplicated new-record and existing-record logic paths.
5. Flow 1 treats extraction uncertainty the same as an explicit failed pre-screen response.
6. Some flow mappings may write counter values as strings even though the schema defines numbers.
7. Timestamp expression syntax should be verified to prevent literal `$now()` strings.
8. Flow 2 does not automatically cancel an unmatched Calendly event.
9. Flow 4 and Auto-Dialer currently use different configured time zones.
10. The Driver Application Analysis agent has an empty system prompt.
11. Flow 7's missing-data priority currently checks only phone and email.
12. Tobacco policy should remain aligned across Flow 1, Riley, Flow 5, and recruiter policy.

---

## 24. Recommended Future-State Improvements

- Consolidate Flow 1's duplicate create/update branches.
- Add explicit pre-screen `needs_review` handling for extraction ambiguity.
- Correct all candidate-facing email recipients.
- Normalize all counters as numeric values.
- Standardize scheduled-flow time zones.
- Repair the Flow 5 to Flow 6 callback-counter handoff.
- Upgrade Flow 6 to `cronExpressions`.
- Add explicit API error branches and operational alerts.
- Add a controlled enum definition for workflow state fields.
- Add automated schema-to-flow validation for field names and types.
- Expand Flow 7 data-quality checks beyond phone and email.
- Add dashboard reporting for conversion, abandonment, recovery, and recruiter aging.

---

## 25. Maintenance Rules

Whenever a production flow or agent changes:

1. Export the published JSON.
2. Confirm `version` and `publishedVersion` inside the JSON.
3. Review changed field mappings and state transitions.
4. Update this architecture document.
5. Update the known-deviations section.
6. Validate the Data Store schema.
7. Confirm that candidate and recruiter email recipients are correct.
8. Confirm that all timestamp expressions evaluate correctly.
9. Confirm field ownership for counters and statuses.
10. Record the change in repository history.
