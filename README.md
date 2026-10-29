# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Maryam Noman
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** about 2 to 2.5 hours

## What this is

A design for an AI-assisted proposal triage tool. AI extracts information from proposals, code handles eligibility checks, scoring and ranking, and a human makes the final award decision. No application was built.

## Files

| File | Description |
|---|---|
| [WRITEUP.md](WRITEUP.md) | Answers to the five design questions |
| [diagram/flow.png](diagram/flow.png) | Workflow showing AI steps, rule-based checks and human review |
| [PROMPTS.md](PROMPTS.md) | Prompts for evidence extraction and the team checklist |
| [ranking.json](ranking.json) | Expected ranking and status of proposals A to E |
| [calculations/](calculations/) | Supporting hand-written scoring calculations |

## How I used AI tools

I used Claude and ChatGPT as assistants to review the design, refine the documentation and examine the scoring assumptions. The final design reflects my own decisions about the scoring method, eligibility checks and handling of exceptions.

### Where I disagreed with AI

- **Experience dates:** the AI first suggested that the extraction prompt should judge "last 5 years" using today's date. I disagreed because proposals A, E and C only say "since 2021" or "since 2020", so every one of them would be marked unclear and the ranking would break. I decided that the AI should only copy the experience period as stated in each proposal, and code handles the five-year window.
- **Monitoring plan:** the AI suggested giving C no credit for monitoring, because weekly progress reports are not system performance monitoring. I chose to count regular progress reporting as a monitoring plan, because weekly reports do monitor the progress of the work. I documented this assumption in `WRITEUP.md`, along with the alternative reading (C would score 86.03 and the ranking would not change).
