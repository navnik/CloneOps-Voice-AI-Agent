# Prompt Variables

This file documents runtime variables that CloneOps may inject into Riley's system prompt or conversation context.

## Candidate identity

| Variable | Purpose | Required |
|---|---|---:|
| `{{firstName}}` | Candidate first name | Yes |
| `{{lastName}}` | Candidate last name | Recommended |
| `{{candidateName}}` | Full candidate name | Recommended |
| `{{phoneNumber}}` | Outbound phone number | Recommended |
| `{{email}}` | Candidate email | Recommended |

## Candidate location

| Variable | Purpose | Required |
|---|---|---:|
| `{{city}}` | Candidate city from application | Recommended |
| `{{state}}` | Candidate state from application | Recommended |

## Application context

| Variable | Purpose | Required |
|---|---|---:|
| `{{applicationThreadId}}` | Source application thread / data store key | Yes |
| `{{applicationSubmittedAt}}` | Date/time application was submitted | Recommended |
| `{{leadSource}}` | Source of application | Optional |
| `{{professionalBackground}}` | Candidate-provided background | Optional |
| `{{comments}}` | Candidate comments from application | Optional |

## Pre-screen answers

| Variable | Purpose |
|---|---|
| `{{minimumAgeQualified}}` | Age qualification answer from application |
| `{{drivingExperienceQualified}}` | Driving experience qualification answer from application |
| `{{englishFluent}}` | English fluency answer from application |
| `{{workAuthorized}}` | Work authorization answer from application |
| `{{hasSmartphone}}` | Smartphone answer from application |
| `{{usesTobacco}}` | Tobacco answer from application |

## Scheduling context

| Variable | Purpose |
|---|---|
| `{{scheduledCallTime}}` | Calendly scheduled screening time |
| `{{calendlyInviteeUri}}` | Calendly invitee URI |
| `{{calendlyEventUri}}` | Calendly event URI |

## Telephony and workflow

| Variable | Purpose |
|---|---|
| `{{ivr_task}}` | IVR navigation task if CloneOps supports IVR tools |
| `{{callbackEnabled}}` | Whether callback capture is enabled |
| `{{voicemailEnabled}}` | Whether voicemail should be left |

## Guidance

Do not require Riley to ask for every variable. Runtime variables are context, not a checklist. Riley should verify high-risk information naturally and ask only when the information is needed for the active conversation topic.
