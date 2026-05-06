# Idea Intake & Deduplication System — Spec v0.2

**Status:** Draft
**Owner:** TBD
**Last updated:** 2026-05-05

---

## Problem

Users and their AI agents start work on ideas without checking for existing or in-flight effort. Result: duplicated work, wasted token spend, fragmented ownership.

## Goals

- Capture ideas at the moment of ideation, before work begins.
- Detect duplicate or complementary effort and surface it to the user.
- Centralize idea intake with traceable metadata in a pluggable storage backend.
- Ship as an open-source Skill/solution so any team can plug in their own system of record (SoR).

## Non-Goals (this version)

- Running the review/approval process for novel ideas (external; nice-to-have hook only).
- Executing approved ideas.
- Reviewer tooling.

## Actors

- **Submitter** — user, typically working through an AI agent.
- **Submitter's Agent** — Claude instance with the Skill installed; generates the bare-bones spec and submits on the user's behalf.
- **Auditor / Tracker** — queries intake history.
- **Operator** — team deploying the system; selects and configures the SoR adapter.

## Conceptual Flow

```
User ideating in Claude
        │
        ▼
┌────────────────────────┐
│ Skill activates        │  proactive (detected ideation)
│                        │  OR manual invoke
└──────────┬─────────────┘
           │ generates bare-bones spec
           ▼
┌────────────────────────┐         ┌─────────────────────────┐
│ Match Engine           │ ──────▶ │ SoR Adapter (interface) │
│  - auto-flag candidates│         │   ├─ Notion (reference) │
│  - human confirms      │ ◀────── │   ├─ Airtable           │
└──────────┬─────────────┘         │   ├─ Google Sheets      │
           │                       │   ├─ Postgres / SQLite  │
           │                       │   └─ ...your backend    │
           │                       └─────────────────────────┘
           ├─ Match found  → present to user → user picks:
           │                   • abandon
           │                   • link as related
           │                   • proceed w/ justification (logged)
           │
           └─ Novel        → submitted, status = Pending Review
                            (review process external, out of scope)
```

## Functional Requirements

### Intake

- **FR1** — Skill activates two ways: (a) proactive — detects user is ideating/prototyping; (b) manual invoke.
- **FR2** — Skill generates a bare-bones spec from the conversation. *(Template — deferred.)*
- **FR3** — Skill submits the spec on the user's behalf.

### Matching

- **FR4** — System compares submission against prior submissions (primary). Secondary: active work in PM systems (e.g., Linear, Jira) — Phase-2.
- **FR5** — Hybrid match: system auto-flags candidates above a similarity threshold; user confirms whether each is `duplicate`, `complementary`, or `not a match`.

### User Action on Match

- **FR6** — When a match is confirmed, submitter chooses one of:
  - `abandon`
  - `link as related`
  - `proceed w/ justification` (justification captured and logged)

  This is also the dispute path for users who disagree with the auto-flag.

### Persistence (Storage-Agnostic)

- **FR7** — All submissions persist to a configured **SoR adapter**. Every submission is logged, including those abandoned post-match and those proceeded-with-justification.
- **FR8** — Required metadata per submission (storage-agnostic):
  - title
  - summary
  - submitter
  - department
  - business area
  - timestamp
  - source agent
  - priority
  - estimated effort
  - related OKR / initiative
  - requested-by
  - status (`Pending Review` | `Linked` | `Abandoned` | `Proceeded-w/-Justification`)
- **FR9** — All users can view all submissions (read access through the same adapter).

### Storage Adapter Contract

- **FR12** — System defines a minimal **adapter interface** that any backend must implement:
  - `submit(record) → id`
  - `get(id) → record`
  - `list(filter) → [records]`
  - `update_status(id, status) → ok`
  - `search(query) → [candidate matches]` *(used by Match Engine; backend may delegate to native search or fall back to a generic semantic search layer)*
  - `comment(id, text) → ok` *(optional; if backend doesn't support, system falls back to next notification channel)*
- **FR13** — Reference adapter ships for Notion. Additional adapters (which ones — see deferred) are community-extensible.
- **FR14** — Adapter selection is configuration-driven; swapping the SoR must not require changes to the Skill or Match Engine.

### Routing

- **FR10** — Novel submissions (no match, or match dismissed) are auto-set to status `Pending Review` via the adapter. The downstream review process is external and out of scope; the system only marks status and stops.

### Notification

- **FR11** — Verdict delivery, in priority order:
  1. Agent in-chat reply — primary, when via Skill
  2. SoR record annotation (e.g., Notion page comment, Airtable comment, DB audit row) + status update — always, if adapter supports it
  3. Email — optional / secondary
  4. Slack / Teams — optional / secondary

## Non-Functional Requirements

- **NFR1** — Visibility: open to all users (subject to adapter's own access controls).
- **NFR2** — SoR downtime behavior — *deferred*.
- **NFR3** — Expected volume — *deferred*.
- **NFR4** — Skill must be distributable enterprise-wide to Claude instances.
- **NFR5** — Open-source-ready: adapter interface is documented; adding a new backend should not require changes to core code.
- **NFR6** — Secrets/config handling: adapter credentials live in environment/config, never in the Skill itself.

## Deferred / Open

Low-leverage items to settle before `plan.md`:

- Bare-bones spec template — what fields the Skill generates.
- Resubmission / versioning — if user revises after a match, new entry or update?
- Proactive-trigger heuristic — what signals "user is ideating"? (false-positive tolerance)
- Similarity threshold — value and tunability.
- Phase-2 PM-system match scope (Linear / Jira specifics).
- Adapter portfolio for v1 — which backends ship out of the box vs. community-contributed.
- Auth / identity mechanics for the Skill → adapter path.
- Retention policy.
- How the system handles `search` for adapters with weak native search (fallback embedding/index layer?).

## Decision Log

Captured during interview:

| # | Question | Decision |
|---|----------|----------|
| Q1 | Match determination | Hybrid: auto-flag, human confirms |
| Q2 | Action on match | Choose: abandon / link / proceed w/ justification |
| Q3 | Review process | Out of scope; mark `Pending Review` only |
| Q4 | Required metadata | Full set (title, summary, submitter, dept, business area, timestamp, source agent, priority, effort, OKR, requested-by) |
| Q5 | Distribution model | Claude Skill (proactive ideation detection) |
| Q6 | Notification | Agent-first → SoR comment/status → email/Slack optional |
| Q7 | Dispute path | Collapsed into Q2 (`proceed w/ justification`) |
| Q8 | Visibility | All users see all ideas |
| Q9 | Match scope | Prior submissions (primary), Linear/Jira (Phase-2) |
| Q10 | Skill activation | Proactive + manual invoke |
| Q11 | Storage model | Pluggable SoR adapter; Notion is reference impl; OSS-ready |

## Changelog

- **v0.2** — Reframed storage as pluggable SoR adapter (FR7–FR14, NFR5–NFR6). Notion demoted to reference implementation. Added Operator actor.
- **v0.1** — Initial draft.
