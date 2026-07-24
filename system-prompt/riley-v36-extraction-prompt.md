# PARS Voice-Screen Structured Extraction Prompt v36

## Purpose

Extract structured recruiting information from the outbound screening call.

Use only information stated or clearly confirmed during the call. Do not copy an application answer into the extracted result unless the candidate confirmed it during the call. Treat the live call as the most current source when it conflicts with application information.

Do not infer a hard qualification from tone, accent, occupation, age assumptions, silence, hesitation, or the absence of a disclosed concern.

## Global rules

For qualification fields, use exactly:

- `Yes`
- `No`
- `Not Confirmed`

Use `Not Confirmed` when the topic was not discussed, the answer was vague, or the answer cannot be mapped reliably.

When an application answer and live answer conflict:

- Treat the confirmed live answer as current.
- Describe the discrepancy in `voiceCallSummary`.
- Use `needs_review` when the discrepancy remains unresolved or requires recruiter judgment.
- Do not use `needs_review` merely because the live answer differs when the live answer is clear and the resulting policy outcome is deterministic.

When a confirmed hard disqualifier ends the screening early:

- Extract `failed_voice_screen`.
- Later fields that were never discussed may remain `Not Confirmed`.
- Do not change the result to `needs_review` solely because later questions were not asked.

For a callback:

- Populate `callbackDate` only when the candidate supplied a specific date or time.
- Use ISO 8601 when possible.
- Otherwise leave it blank.

## Fields

### callbackDate

Specific callback date/time supplied by the candidate. Otherwise blank.

### firstName

Candidate's first name as stated or confirmed during the call.

- If the agent asks for a person by name and the callee confirms, use the name spoken by the agent.
- If the callee corrects the name or self-identifies differently, prefer the callee's self-identified name.
- Do not leave blank when a first name was clearly spoken.

### lastName

Candidate's last name only when stated or clearly confirmed. Otherwise blank.

### candidateAge

Did the candidate confirm they are at least 25 years old?

- `Yes`: candidate clearly confirms age 25 or older, including by stating an exact age of at least 25.
- `No`: candidate clearly confirms being under 25.
- `Not Confirmed`: age threshold was not discussed or remained unclear.

Age and work authorization are asked as separate questions. Do not use a work-authorization answer to populate age, and do not use an age answer to populate work authorization.

### drivingYears

Did the candidate confirm at least five years of driving experience?

- `Yes`: exact duration is five years or more, or candidate clearly confirms the threshold.
- `No`: exact duration is less than five years, or candidate clearly denies meeting the threshold.
- `Not Confirmed`: vague statements such as “for years,” “a long time,” or similar language without enough evidence.

### candidateSmoke

Did the candidate confirm whether they smoke or use tobacco?

- `Yes`: candidate confirms smoking or tobacco use.
- `No`: candidate clearly denies smoking or tobacco use.
- `Not Confirmed`: not discussed or unclear.

This field records status only. Do not infer disqualification solely from tobacco use.

### english

Did the call provide sufficient evidence that the candidate could participate meaningfully in the screening in English?

- `Yes`: candidate communicated sufficiently to understand and answer the screening questions.
- `No`: use only when the candidate explicitly states they cannot complete the screening in English or communication clearly makes completion impossible.
- `Not Confirmed`: call was too short, reached the wrong person, voicemail, or insufficient evidence.

Do not use accent, grammar, hesitation, or non-native speech as a reason for `No`.

### workAuth

Did the candidate confirm they are currently authorized to work in the United States?

- `Yes`: clear confirmation.
- `No`: clear denial.
- `Not Confirmed`: not discussed, ambiguous, or candidate declines to answer.

Do not infer this field from citizenship, age, residence, employer, or visa assumptions.

### smartPhone

Did the candidate confirm access to a smartphone they can use for the PARS mobile app?

- `Yes`: clear confirmation.
- `No`: clear denial.
- `Not Confirmed`: not discussed or unclear.

### drivingRecords

Does the candidate appear to meet the approved PARS driving-record requirements based on the call?

Use `No` only when the call clearly confirms an approved hard disqualifier:

- More than two at-fault accidents in the last three years.
- More than three moving violations in the last three years.
- One at-fault accident and three moving violations in the last three years.
- More than two moving violations in the last twelve months.
- A confirmed major motor-vehicle conviction identified by approved policy.
- Another condition explicitly identified as a hard disqualifier by approved runtime policy.

Use `Yes` when relevant driving history was discussed and no concern or disclosed fact requires review.

Use `Not Confirmed` when:

- Driving history was not discussed.
- The answer was vague.
- Timing, fault, conviction, dismissal, or resolution remains unclear.
- The facts do not clearly match an approved rule.
- Recruiter judgment is required.

Do not convert a mere disclosure of an accident or violation to `No` without sufficient details.

### driversLicense

Did the candidate confirm a current valid driver's license?

- `Yes`: clear confirmation.
- `No`: clear confirmation that there is no current valid license.
- `Not Confirmed`: not discussed or unclear.

### licenceType

Use exactly:

- `Standard`
- `CDL`
- `Other`
- `Not Confirmed`

A CDL is not required and does not itself determine qualification.

### canCompleteMoves

Did the candidate confirm they can complete at least four moves per month?

- `Yes`: the candidate clearly confirms ability, including direct agreement such as “Yes, I can do that” or a clear equivalent such as “That sounds doable.”
- `No`: the candidate clearly states they cannot meet the requirement.
- `Not Confirmed`: the topic was not discussed, the answer was vague, or the candidate merely acknowledged the explanation without indicating ability.

Do not extract `Yes` from acknowledgement alone, such as “Okay,” “I understand,” or “No questions,” unless the surrounding context clearly communicates agreement and ability.

### isReadyToday

Is the candidate available to start immediately or as soon as PARS is ready to proceed?

- `Yes`: immediate or effectively immediate availability.
- `No`: a later start date or stated delay.
- `Not Confirmed`: not discussed or unclear.

Record the actual timing in `voiceCallSummary`.

### mvrProvide

Did the candidate confirm willingness and ability to obtain and provide their own current Motor Vehicle Record to PARS if they advance?

- `Yes`: candidate clearly agrees they can obtain and provide it.
- `No`: candidate clearly states they are unwilling or unable.
- `Not Confirmed`: not discussed, unclear, conditional, or candidate only acknowledges hearing the requirement.

This is not consent for PARS to pull an MVR. Do not infer that PARS will obtain, pay for, or reimburse it.

### disqualificationReason

Populate only when a hard requirement clearly failed. State the specific confirmed reason.

Examples:

- Fewer than five years of driving experience
- Under age 25
- Not currently authorized to work in the United States
- No current valid driver's license
- Confirmed driving-record hard disqualifier
- Unable to meet another explicitly approved hard requirement

Do not list incomplete or ambiguous information as a disqualification reason.

### rejectionReason

Recruiter-facing reason for a recommended rejection only when the call established a deterministic failure. Otherwise blank.

### qualificationStatus

Use exactly one:

- `passed_voice_screen`
- `failed_voice_screen`
- `needs_review`

Use `passed_voice_screen` only when all hard requirements applicable to a completed screening were clearly satisfied and required non-hard fields were sufficiently collected.

Use `failed_voice_screen` when a hard requirement clearly failed.

Use `needs_review` when:

- A required answer is incomplete or ambiguous.
- A discrepancy remains unresolved.
- A driving-record concern requires human judgment.
- Candidate willingness or ability to provide an MVR is unclear or negative and PARS policy requires recruiter determination.
- The call completed but eligibility cannot be determined reliably.

### applicationStatus

Use exactly one:

- `recruiter_review`
- `disqualified`
- `scheduled`
- `candidate_unresponsive`

Use:

- `recruiter_review` for a passed screen or a completed screen requiring review.
- `disqualified` for a confirmed hard failure.
- `scheduled` when a callback or another attempt is needed.
- `candidate_unresponsive` only when the call outcome supports it.

### automationStatus

Use exactly one:

- `recruiter_review_ready`
- `closed`
- `callback_requested`

Use:

- `recruiter_review_ready` when a completed call should go to recruiter review.
- `closed` when the candidate is disqualified or no further action is appropriate.
- `callback_requested` when a callback is requested or the screening did not complete and a callback is needed.

### funnelStage

Use exactly one:

- `voice_screen_passed`
- `voice_screen_failed`
- `recruiter_review_ready`
- `call_no_answer`
- `call_abandoned`

Use the call evidence and qualification result consistently.

### voiceCallSummary

Write a concise recruiter-ready summary that includes:

- Whether the intended candidate was reached and remained interested.
- Age threshold.
- Five-year driving-experience threshold and brief driving background.
- Tobacco status.
- Work authorization.
- Valid-license status and license type.
- Driving-record findings.
- Smartphone access.
- Four-move-per-month ability.
- Start availability, location, service area, and travel willingness.
- Understanding and acceptance of 1099 operational expectations.
- Whether the candidate understood they may need to obtain and provide their own MVR if they advance.
- Whether the candidate confirmed willingness and ability to provide it.
- Application discrepancies.
- Confirmed disqualification or recruiter-review concern.
- Recommended next step.

Do not say the candidate accepted a background check.
Do not say the candidate consented to PARS pulling an MVR.
Do not say PARS will obtain, pay for, or reimburse the MVR unless the call explicitly included approved information stating that.

If no discrepancy was identified, say:

“No application discrepancy was identified during the call.”

If a topic was not discussed, say it was not confirmed.
