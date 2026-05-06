# Unbubble-AI
 
> Catch duplicate work before it starts. AI-powered idea intake and deduplication, with a pluggable system of record.
 
**Status:** 🟡 Pre-alpha — spec stage. No implementation yet. See [`spec.md`](./spec.md) for the v0.2 draft.
 
---
 
## Why this exists
 
Users and their AI agents start work on ideas without checking for existing or in-flight effort. The result is duplicated work, wasted token spend, and fragmented ownership.
 
Unbubble-AI sits at the moment of ideation — *before* the agent starts building — and answers one question:
 
> "Has someone (or some agent) already started on this?"
 
If yes, you get the existing work surfaced and decide what to do. If no, the idea is logged with full provenance and routed onward.
 
## What it does
 
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
           ├─ Match found  → user picks:
           │                   • abandon
           │                   • link as related
           │                   • proceed w/ justification (logged)
           │
           └─ Novel        → submitted, status = Pending Review
```
 
## How it works
 
1. **The Skill activates** — proactively when it detects a user is ideating or prototyping, or on manual invoke.
2. **It generates a bare-bones spec** from the conversation context.
3. **The Match Engine searches the system of record** for prior submissions above a similarity threshold.
4. **The user confirms** whether each candidate is a duplicate, complementary, or unrelated.
5. **The submission persists** with full metadata (submitter, department, business area, OKR, source agent, etc.) regardless of outcome — including abandoned and proceeded-with-justification cases.
## Storage adapter model
 
Unbubble-AI does not hard-code a backend. It defines a minimal adapter interface — `submit`, `get`, `list`, `update_status`, `search`, optional `comment` — and any storage system that implements it can serve as the system of record.
 
| Adapter | Status |
|---|---|
| Notion | Reference implementation (planned v1) |
| Airtable | Community-extensible |
| Google Sheets | Community-extensible |
| Postgres / SQLite | Community-extensible |
| Your backend | Implement the interface |
 
Adapter selection is configuration-driven. Swapping the backend does not require changes to the Skill or Match Engine.
 
> Architectural note: this is the **Ports and Adapters** (a.k.a. **Hexagonal**) pattern — the core logic talks to interfaces, not vendors. See [Cockburn's original](https://alistair.cockburn.us/hexagonal-architecture/).
 
## Roadmap
 
- **Phase 1 (v1):** Claude Skill + Notion reference adapter + match against prior submissions.
- **Phase 2:** Cross-system match against active work in PM tools (Linear, Jira).
- **Phase 3+:** Additional reference adapters, configurable similarity thresholds, fallback embedding/index layer for backends with weak native search.
Open items are tracked in [`spec.md` § Deferred / Open](./spec.md#deferred--open).
 
## Project status & non-goals
 
This version explicitly does **not** include:
 
- The downstream review/approval workflow for novel ideas (system marks `Pending Review` and stops).
- Execution of approved ideas.
- Reviewer tooling.
## Contributing
 
Contributions are welcome once v1 lands. For now, the most useful contributions are:
 
- Feedback on [`spec.md`](./spec.md) — open an issue if anything is unclear, missing, or wrong.
- Adapter design proposals — especially how to handle `search` for backends without strong native search.
## License
 
TBD. Candidate licenses under consideration: MIT, Apache 2.0. *(Swap once decided.)*
 
---
