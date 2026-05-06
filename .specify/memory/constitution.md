# Speckit Constitution — Unbubble-AI

**Version:** 1.0
**Last updated:** 2026-05-06

> This document governs how specs are written, versioned, reviewed, and retired in this repository. When in doubt, follow the spirit: a spec exists to make decisions legible and traceable, not to create process overhead.

---

## 1. What a spec is (and is not)

A **spec** is a single, authoritative document that answers:

1. What problem are we solving?
2. What will the system do (and deliberately not do)?
3. What decisions were made, and why?

A spec is **not** a design doc, an architecture diagram, a project plan, or a task list. Those live elsewhere (e.g., `plan.md`, ADRs, issue tracker). A spec is the contract between problem space and implementation.

---

## 2. When to write one

Write a spec when:

- You are starting a non-trivial feature or system (more than a day of work, or more than one contributor).
- A significant architectural or product decision needs a record.
- You are contributing a new SoR adapter to this project.

You do not need a spec for:

- Bug fixes.
- Refactors that don't change behavior.
- Documentation-only changes.

---

## 3. File naming and location

| Artifact | Path | Notes |
|---|---|---|
| Primary spec | `spec.md` | One per repository root; this project's core spec |
| Feature sub-specs | `specs/<slug>.md` | For scoped features that warrant their own spec |
| Adapter specs | `specs/adapters/<name>.md` | One per SoR adapter |
| This constitution | `SPECKIT.md` | Repository root; not versioned like a spec |

Slugs are lowercase, hyphen-separated, and derived from the spec title (e.g., `notion-adapter.md`).

---

## 4. Required structure

Every spec must include these sections in this order. Sections marked *(optional)* may be omitted only if genuinely not applicable.

```
# <Title> — Spec v<version>

**Status:** <status>
**Owner:** <name or TBD>
**Last updated:** <YYYY-MM-DD>

---

## Problem
## Goals
## Non-Goals (this version)
## Actors                        ← omit if only one actor
## Conceptual Flow               ← ASCII diagram preferred
## Functional Requirements
## Non-Functional Requirements
## Deferred / Open
## Decision Log
## Changelog
```

### Section guidance

**Problem** — One paragraph. State the pain, not the solution. No bullet points.

**Goals** — What success looks like. Each goal is a bullet. Falsifiable if possible.

**Non-Goals** — What this version explicitly will not do. Saves future arguments.

**Actors** — Named roles that interact with the system. Not org chart titles — behavioral roles.

**Conceptual Flow** — A diagram showing the happy path. ASCII box-and-arrow preferred; keeps diffs readable.

**Functional Requirements** — See §5.

**Non-Functional Requirements** — See §5.

**Deferred / Open** — A flat list of questions not yet answered. Items move out of here as decisions land in the Decision Log.

**Decision Log** — See §6.

**Changelog** — Most recent entry first. See §7.

---

## 5. Requirement numbering

### Functional Requirements (FR)

- Prefix: `FR`
- Numbered sequentially within a spec starting at `FR1`.
- Grouped under named sub-headings (e.g., `### Intake`, `### Matching`).
- Numbering follows *declaration order within a group*, not group order. FR numbers are stable once assigned — renumber only in a major version bump.
- Format: `- **FRn** — <requirement statement.>`

### Non-Functional Requirements (NFR)

- Prefix: `NFR`
- Numbered sequentially within a spec starting at `NFR1`.
- Format: `- **NFRn** — <requirement statement.>`

### Rules

- A requirement states *what*, not *how*.
- One requirement per bullet. Compound requirements must be split.
- Deferred requirements are listed in **Deferred / Open**, not numbered until resolved.
- Requirements are never deleted — they are superseded in a new version or explicitly marked *removed in vX.Y*.

---

## 6. Decision Log

The Decision Log captures every significant decision made during spec development. It is the primary artifact for understanding *why* the spec is the way it is.

### Format

```markdown
| # | Question | Decision |
|---|----------|----------|
| Q1 | <question as asked> | <decision reached> |
```

### Rules

- Log a decision when it resolves an open question or fork in the road.
- The *Question* column states the dilemma, not the answer.
- The *Decision* column states the conclusion only — rationale lives in the commit message or a linked issue.
- Decision numbers (`Q1`, `Q2`, …) are stable. Never renumber.
- Reversed decisions get a new row (e.g., `Q12 — Revisit of Q3`) referencing the original.

---

## 7. Versioning

Specs use a two-part version: `vMAJOR.MINOR`.

| Version range | Meaning |
|---|---|
| `v0.x` | Draft — not yet approved; structure and requirements may change freely |
| `v1.0` | First approved version — requirements are stable |
| `v1.x` | Approved; minor additions or clarifications that don't change existing FRs |
| `v2.0+` | Major revision — existing FRs changed, removed, or substantially rewritten |

**Increment rules:**

- Bump `MINOR` when you add new FRs/NFRs, resolve deferred items, or clarify without reversing prior decisions.
- Bump `MAJOR` when you reverse a decision, remove or substantially rewrite existing FRs, or change the problem statement.
- Never bump the version without updating **Changelog** and **Last updated**.

---

## 8. Status lifecycle

```
Draft → In Review → Approved → Superseded
                ↘ Withdrawn
```

| Status | Meaning |
|---|---|
| `Draft` | Work in progress; not ready for review |
| `In Review` | PR open; reviewers assigned |
| `Approved` | PR merged; spec is the authoritative reference |
| `Superseded` | A newer spec or version replaces this one; link to successor |
| `Withdrawn` | Work cancelled; kept for record; link to reason |

---

## 9. GitHub workflow

### Branch naming

```
spec/<slug>           # new spec
spec/<slug>-vX.Y      # revision to existing spec
```

Examples: `spec/notion-adapter`, `spec/idea-intake-v1.0`

### Pull request

- Title: `spec: <title> vX.Y`
- PR description must include: what changed since the last version (or "initial draft"), and a link to any related issues.
- At least one review required before merge to `main` (waived for `v0.x` drafts if the owner self-approves and logs it).
- Merge strategy: squash. The squash commit message is the canonical record of the review decision.

### Review checklist

Before approving a spec PR, a reviewer confirms:

- [ ] All required sections are present and in order.
- [ ] Every FR and NFR is numbered correctly and states *what*, not *how*.
- [ ] The Decision Log captures all forks resolved during drafting.
- [ ] Deferred items are listed explicitly; nothing is silently assumed.
- [ ] Changelog entry added; version and date updated.
- [ ] No implementation detail leaks into requirements (no class names, endpoint paths, etc.).

---

## 10. Amending an approved spec

Never edit an approved spec's existing FRs in-place without a version bump. The workflow:

1. Branch off `main`: `spec/<slug>-vX.Y`.
2. Make changes; bump version; update Changelog.
3. Open PR with diff summary in the description.
4. Review and merge.

For typo fixes or non-semantic clarifications: bump `MINOR`, note in Changelog as "editorial".

---

## 11. Retiring a spec

When a spec is no longer the active reference:

- Set `**Status:** Superseded` (or `Withdrawn`).
- Add a top-of-file note: `> Superseded by [spec/new-thing.md](./specs/new-thing.md) as of vX.Y.`
- Do not delete the file. History is valuable.

---

## 12. Spec template

Copy this to start a new spec:

```markdown
# <Title> — Spec v0.1

**Status:** Draft
**Owner:** TBD
**Last updated:** YYYY-MM-DD

---

## Problem

<!-- One paragraph. State the pain. -->

## Goals

- 

## Non-Goals (this version)

- 

## Actors

- **<Role>** — <description>

## Conceptual Flow

​```
<!-- ASCII diagram -->
​```

## Functional Requirements

### <Group name>

- **FR1** — 

## Non-Functional Requirements

- **NFR1** — 

## Deferred / Open

- 

## Decision Log

| # | Question | Decision |
|---|----------|----------|

## Changelog

- **v0.1** — Initial draft.
```

---

## Changelog

- **v1.0** — Initial constitution.
