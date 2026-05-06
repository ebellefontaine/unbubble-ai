# Feature Specification: Idea Intake & Deduplication System

**Feature Branch**: `001-idea-intake-deduplication`
**Created**: 2026-05-05
**Status**: Draft

## User Scenarios & Testing _(mandatory)_

### User Story 1 — Submitter captures a new idea (Priority: P1)

A user is ideating in Claude. The Skill detects ideation in progress (or the user invokes it manually), generates a bare-bones spec from the conversation, and submits it to the configured system of record. The user receives an in-chat confirmation with the submission ID and status.

**Why this priority**: This is the core intake path. Without it, nothing else in the system functions.

**Independent Test**: Can be fully tested by starting a Claude session with the Skill installed, describing a new idea, and verifying that a record with the correct metadata appears in the SoR with status `Pending Review`.

**Acceptance Scenarios**:

1. **Given** a user is describing a new idea in Claude and the Skill is installed, **When** the Skill detects ideation, **Then** it generates a bare-bones spec and submits it to the configured SoR adapter without requiring the user to manually trigger it.
2. **Given** a user manually invokes the Skill, **When** the Skill runs, **Then** it generates a spec from the current conversation and submits it to the SoR adapter.
3. **Given** a submission is made, **When** the SoR adapter persists it, **Then** the record includes all required metadata fields (title, summary, submitter, department, business area, timestamp, source agent, priority, estimated effort, related OKR/initiative, requested-by, status).
4. **Given** a submission succeeds, **When** the agent replies, **Then** the user receives an in-chat confirmation with the submission ID and a status of `Pending Review`.

### User Story 2 — Submitter handles a duplicate match (Priority: P1)

After submitting an idea, the Match Engine finds one or more prior submissions above the similarity threshold. The system presents the candidates to the user, who confirms whether each is a duplicate, complementary, or not a match. Based on the user's verdict, the submission is updated accordingly.

**Why this priority**: Deduplication is the primary value proposition. A system that accepts every submission without checking for overlap defeats the purpose.

**Independent Test**: Can be fully tested by seeding the SoR with an existing submission, then submitting a semantically similar idea and verifying that the candidate is surfaced and the user's verdict is applied correctly.

**Acceptance Scenarios**:

1. **Given** a new submission is made, **When** the Match Engine finds prior submissions above the similarity threshold, **Then** those candidates are presented to the user before the submission is finalised.
2. **Given** a candidate is presented, **When** the user confirms it as a `duplicate`, **Then** the new submission status is set to `Abandoned` and the user is shown the existing submission.
3. **Given** a candidate is presented, **When** the user marks it as `complementary`, **Then** the new submission status is set to `Linked` and both records are updated with a cross-reference.
4. **Given** a candidate is presented, **When** the user marks it as `not a match`, **Then** the match is dismissed and the new submission proceeds to `Pending Review`.
5. **Given** multiple candidates are returned, **When** the user reviews them, **Then** each is handled independently before the submission is finalised.

### User Story 3 — Submitter proceeds with justification despite a match (Priority: P1)

The user disagrees with a duplicate match or believes the new idea is sufficiently distinct to warrant proceeding. They choose "proceed with justification", provide a reason, and the submission is logged with that justification attached.

**Why this priority**: Without this escape hatch the system becomes a blocker rather than a guardrail, and users will route around it.

**Independent Test**: Can be fully tested by triggering a duplicate match, choosing "proceed with justification", entering a reason, and verifying the submission is logged with status `Proceeded-w/-Justification` and the justification text attached.

**Acceptance Scenarios**:

1. **Given** a duplicate match is surfaced, **When** the user selects "proceed with justification", **Then** the system prompts for a free-text justification before finalising.
2. **Given** the user provides a justification, **When** the submission is saved, **Then** the status is `Proceeded-w/-Justification` and the justification text is stored on the record.
3. **Given** the submission is finalised with a justification, **When** the SoR adapter supports comments, **Then** the justification is also written as a comment on the SoR record.

### User Story 4 — Operator configures a new SoR adapter (Priority: P2)

An operator deploys Unbubble-AI for their team and wants to use their own storage backend. They implement the adapter interface, point the configuration at their implementation, and the Skill and Match Engine work without any changes.

**Why this priority**: OSS-extensibility is a stated goal, but the core intake and matching flows (P1) must work first.

**Independent Test**: Can be fully tested by implementing a minimal in-memory adapter that satisfies the interface, setting it as the configured adapter, and running the full User Story 1 and 2 flows against it.

**Acceptance Scenarios**:

1. **Given** a custom adapter that implements `submit`, `get`, `list`, `update_status`, `search`, **When** it is set as the configured adapter, **Then** the Skill and Match Engine operate without code changes.
2. **Given** a custom adapter that does not implement `comment`, **When** a verdict is delivered, **Then** the system falls back to the next notification channel without erroring.
3. **Given** the adapter credentials are stored in environment variables, **When** the adapter initialises, **Then** no credential appears in source code or the Skill definition.

### User Story 5 — Auditor queries submission history (Priority: P2)

A team member wants to browse or search all prior submissions to understand the idea landscape before starting new work.

**Why this priority**: Visibility is a non-functional requirement but secondary to the intake and match flows.

**Independent Test**: Can be fully tested by calling `list()` and `search()` on the adapter directly and verifying that all submissions — including abandoned and proceeded-with-justification ones — are returned.

**Acceptance Scenarios**:

1. **Given** submissions exist with various statuses, **When** an auditor calls `list()`, **Then** all submissions are returned regardless of status.
2. **Given** an auditor provides a search query, **When** `search()` is called, **Then** semantically relevant submissions are returned ranked by similarity.

---

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST activate the Skill proactively when it detects a user is ideating or prototyping in a Claude session.
- **FR-002**: System MUST support manual Skill invocation as an alternative to proactive activation.
- **FR-003**: System MUST generate a bare-bones spec from the current conversation context and submit it to the configured SoR adapter on the user's behalf.
- **FR-004**: System MUST compare each new submission against all prior submissions using a hybrid match strategy: auto-flagging candidates above a configurable similarity threshold, then requiring human confirmation.
- **FR-005**: System MUST present matched candidates to the user and collect a verdict of `duplicate`, `complementary`, or `not a match` for each.
- **FR-006**: System MUST support three user actions on a confirmed match: `abandon` (sets status `Abandoned`), `link as related` (sets status `Linked` and cross-references both records), and `proceed with justification` (captures free-text justification and sets status `Proceeded-w/-Justification`).
- **FR-007**: System MUST persist every submission to the configured SoR adapter regardless of outcome, including abandoned and proceeded-with-justification cases.
- **FR-008**: System MUST store the following metadata on every submission: title, summary, submitter, department, business area, timestamp, source agent, priority, estimated effort, related OKR/initiative, requested-by, status.
- **FR-009**: System MUST allow all users to read all submissions through the SoR adapter.
- **FR-010**: System MUST set novel submissions (no match confirmed, or all matches dismissed) to status `Pending Review` and stop; the downstream review process is out of scope.
- **FR-011**: System MUST deliver the verdict in this priority order: (1) agent in-chat reply, (2) SoR record annotation and status update if the adapter supports it, (3) email if configured, (4) Slack/Teams if configured.
- **FR-012**: System MUST define a minimal adapter interface — `submit(record) → id`, `get(id) → record`, `list(filter) → [records]`, `update_status(id, status) → ok`, `search(query) → [candidates]`, `comment(id, text) → ok` (optional) — that any backend must implement.
- **FR-013**: System MUST ship a reference SoR adapter for Notion. Additional adapters are community-extensible.
- **FR-014**: System MUST select the active adapter through configuration alone; swapping the SoR MUST NOT require changes to the Skill or Match Engine.

### Key Entities

- **Submission**: The primary record. Holds all required metadata fields plus status. Immutable once written except for status and comment updates through the adapter interface.
- **SoR Adapter**: Any backend that implements the six-method interface (`submit`, `get`, `list`, `update_status`, `search`, `comment`). The adapter is the only component that knows about the underlying storage technology.
- **Match Candidate**: A prior submission returned by `search()` above the similarity threshold. Carries its similarity score and the fields needed to present it to the user.
- **Verdict**: The user's decision on a match candidate — `duplicate`, `complementary`, or `not a match`. Drives the status transition and cross-reference logic.
- **Skill**: The Claude agent component. Handles proactive detection, spec generation, submission, match presentation, verdict collection, and in-chat notification.
- **Match Engine**: The component that calls `search()`, applies the threshold, and orchestrates the human-confirmation loop. Decoupled from the Skill's conversational layer.

---

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: A new idea submitted through the Skill appears in the SoR with all required metadata fields populated within one conversational turn.
- **SC-002**: A semantically duplicate submission is flagged as a match candidate before the user's idea is logged as novel.
- **SC-003**: Swapping the configured SoR adapter from Notion to a community adapter requires zero changes to the Skill or Match Engine source.
- **SC-004**: Every submission — including abandoned and proceeded-with-justification cases — is retrievable via `list()` with its full metadata and final status.
- **SC-005**: An operator can deploy the system with a new adapter by implementing the six-method interface and setting a configuration value, without reading or modifying core code.

---

## Assumptions

- The bare-bones spec template (the specific fields the Skill extracts from a conversation) is deferred and will be defined before `plan.md` is written.
- Resubmission and versioning behaviour (new entry vs. update when a user revises after a match) is deferred.
- The proactive-trigger heuristic (what conversational signals indicate ideation) and its false-positive tolerance are deferred.
- The similarity threshold value and whether it is user-tunable are deferred.
- Phase-2 PM-system matching scope (Linear, Jira specifics) is out of scope for this version.
- Which adapter backends ship out of the box beyond Notion is deferred to the adapter portfolio decision.
- Auth and identity mechanics for the Skill-to-adapter path are deferred.
- Retention policy is deferred.
- Behaviour when the SoR is unavailable is deferred.
- For adapters with weak native search, a fallback embedding/index layer may be needed; the design of that layer is deferred.
- Expected submission volume is unknown; no throughput or latency SLAs are set for this version.
