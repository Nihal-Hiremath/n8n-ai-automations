# Automated Sales Meeting Prep

An n8n workflow that researches every external meeting on a sales rep's calendar and emails them a one-page prep brief before the day starts.

## Problem

Sales reps walk into calls without context. Checking the CRM, the prospect's website and past notes takes 10 to 15 minutes per meeting, and on busy days it gets skipped.

## How it works

```mermaid
flowchart LR
    A[Schedule<br/>Weekdays 7 AM] --> B[Google Calendar<br/>today's events]
    B --> C[Filter external<br/>attendees]
    C --> D[HubSpot<br/>contact lookup]
    D --> E[Company website<br/>homepage text]
    E --> F[OpenAI<br/>structured brief]
    F --> G[Gmail<br/>brief to rep]
```

1. **Find meetings.** Pulls today's calendar events and keeps only meetings with at least one attendee outside the company domain.
2. **Research.** Looks up the attendee in HubSpot by email and fetches the company homepage. Personal email domains (Gmail, GMX, etc.) skip the website step.
3. **Generate.** Sends meeting details, CRM data and website text to OpenAI with a strict JSON output format.
4. **Deliver.** Formats the brief as HTML and emails it to the rep.

Each brief contains: summary, company overview, person overview, relationship history, talking points, questions to ask, and risks.

## Reliability

- CRM lookup and website fetch continue on error. A missing record or dead site produces a thinner brief, not a failed run.
- HubSpot and OpenAI calls retry up to 3 times.
- The prompt tells the model to use only provided data and say "unknown" instead of guessing.
- Website text is capped (default 4,000 characters) to keep token cost predictable.

## Setup

1. Import `workflow.json` into n8n (Workflows > Import from File).
2. Open the **Config** node and set:

| Field | Example | Purpose |
|---|---|---|
| `internalDomain` | `yourcompany.com` | Attendees on this domain are ignored |
| `repEmail` | `you@yourcompany.com` | Where the brief is sent |
| `personalEmailDomains` | `gmail.com,gmx.de,...` | Domains with no company website |
| `openaiModel` | `gpt-4o-mini` | Model used for the brief |
| `maxWebsiteChars` | `4000` | Website text limit |
| `timezone` | `Europe/Berlin` | Time shown in the email |

3. Connect credentials:
   - Google Calendar OAuth2
   - HubSpot App Token (private app with `crm.objects.contacts.read`)
   - OpenAI API
   - Gmail OAuth2
4. Run once manually, check the email, then activate.

## Possible extensions

- Add LinkedIn enrichment (Apify or Clay) for the attendee
- Pull recent deal notes and open tickets from HubSpot
- Post the brief to Slack instead of email
- Trigger 30 minutes before each meeting instead of once per morning

## Screenshot

![Workflow](workflow.png)
