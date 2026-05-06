# Unbubble-AI Constitution

## Core Principles

### I. Adapter-First Architecture

Every capability that touches an external system — storage, notifications, PM tools — must go through a defined adapter interface. No vendor-specific code belongs in the core Skill or Match Engine. The Ports and Adapters (Hexagonal) pattern is enforced throughout: core logic talks to interfaces, not vendors. If an implementation choice would require importing a vendor SDK into core, the design is wrong.

### II. Spec-Driven Development

No implementation begins without a spec in `.specify/specs/`. Specs are written before plans; plans are written before tasks. The coding agent must read the relevant spec before generating any plan or code. If a requirement is ambiguous or missing from the spec, surface it as an assumption rather than guessing.

### III. Zero Vendor Lock-in

Swapping the system of record must require only a configuration change — never a code change to the Skill or Match Engine. Adapter credentials live in environment variables or a config file, never in source. The reference adapter (Notion) is an example implementation, not a privileged one.

### IV. OSS-Extensibility by Default

The adapter interface is the primary extension point and must remain documented and stable. Adding a new backend adapter must not require changes to core code. Contributions from outside the core team are expected and welcome; the interface contract is the guarantee they rely on.

### V. Audit Everything

Every submission is logged to the SoR adapter regardless of outcome — including those abandoned post-match and those proceeded-with-justification. There are no silent drops. Provenance (submitter, source agent, timestamp, status) is mandatory on every record.

## Constraints

- The Skill must be distributable enterprise-wide to Claude instances without per-instance code changes.
- The adapter `search` method may delegate to native backend search or a fallback semantic layer — but the interface contract does not change based on backend capability.
- The system marks novel submissions `Pending Review` and stops. It does not own or implement the downstream review/approval workflow.
- All users have read access to all submissions through the same adapter. No per-user visibility filtering in core.

## Development Workflow

- Branches follow the spec-kit convention: `###-feature-name` (e.g., `001-idea-intake-deduplication`).
- Each feature has a spec, plan, and tasks file under `.specify/specs/###-feature-name/`.
- PRs require at least one review before merge to `main`. Squash merge only.
- The coding agent must not introduce breaking changes to the adapter interface without a new spec and version bump.

## Governance

This constitution supersedes all informal agreements, README statements, and prior conventions. Amendments require a PR, a version bump in the footer, and a rationale comment in the PR description. The coding agent must treat this document as the highest-priority reference when resolving ambiguity between the constitution and any other file.

**Version**: 1.0 | **Ratified**: 2026-05-06 | **Last Amended**: 2026-05-06
