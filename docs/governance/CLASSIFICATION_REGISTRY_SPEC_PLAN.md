# Classification Registry Specification Plan

**Status:** DRAFT — Planning Document  
**Created:** 2026-06-25  
**Target:** IEF-Program#11  
**Author:** IEF Coordinator  

This document proposes a formal registry for all IEF classifications used across Program, Operations, Runners, Adapters, and Knowledge domains. It is planning only — no migration, no code changes, no enforcement.

---

## 1. Canonical Classification Naming Rules

### Convention

All classifications use **UPPER_SNAKE_CASE**.

### Namespace Prefixes

Each lifecycle domain gets a namespace prefix:

| Domain | Prefix | Example |
|--------|--------|---------|
| Program (Controller/Coordinator) | `PROG_` | `PROG_ACTION_REQUIRED` |
| Operations (PM/Operator) | `OPS_` | `OPS_WAITING_CODEX` |
| Runners (execution agents) | `RUN_` | `RUN_EXECUTABLE_IDLE` |
| Adapters (external integrations) | `ADP_` | `ADP_SYNC_PENDING` |
| Knowledge (specs, docs) | `KNW_` | `KNW_REVIEW_RETURNED` |
| Cross-domain | `IEF_` | `IEF_BLOCKED`, `IEF_STAGE_READY` |

### Reserved vs User-Extensible Spaces

- Prefixes `PROG_`, `OPS_`, `RUN_`, `ADP_`, `KNW_`, `IEF_` are **reserved** — only Controller or Human Owner may register entries.
- Third-party or project-specific classifications use the `X_` prefix (e.g., `X_CUSTOM_STATUS`). These are not validated by the registry but must not collide with reserved prefixes.

### Naming Constraints

- Maximum length: 64 characters including prefix.
- Allowed characters: `[A-Z0-9_]`.
- Must not begin or end with `_`.
- Must not contain double underscores (`__`).

---

## 2. Lifecycle Stage Binding

### Stage-to-Domain Mapping

| IEF Stage | Primary Domain | Secondary Domains |
|-----------|---------------|-------------------|
| Program (strategy, governance) | `PROG_` | `IEF_` |
| Operations (PM, dispatch, execution) | `OPS_` | `RUN_` |
| Runners (agent execution) | `RUN_` | `OPS_` |
| Knowledge (specs, docs) | `KNW_` | `IEF_` |
| Adapters (external integrations) | `ADP_` | `OPS_` |

### Stage-Specific Classifications

These classifications are meaningful only within their owning stage:

- `RUN_EXECUTABLE_IDLE` — only meaningful in the Runners execution context.
- `KNW_REVIEW_RETURNED` — only meaningful in the Knowledge review context.
- `ADP_SYNC_PENDING` — only meaningful in the Adapters integration context.

### Cross-Stage Classifications

These classifications propagate across stages and require multi-domain awareness:

- `IEF_BLOCKED` / `IEF_UNBLOCKED` — any stage can set; all stages must respect.
- `IEF_STAGE_READY` — signals stage transition; consumed by Program and Operations.
- `IEF_WAITING_HUMAN` — any stage can escalate; resolution is stage-agnostic.

### Stage Transition Triggers

| Transition | Trigger Classification | Direction |
|-----------|----------------------|-----------------|
| Operations to Runners | `OPS_ACTION_REQUIRED` | PM dispatches to Operator |
| Runners to Operations | `RUN_ARTIFACT_DELIVERED` | Operator returns artifact |
| Knowledge to Program | `KNW_REVIEW_RETURNED` | Spec review completed |
| Program to Operations | `PROG_AUTHORIZATION_GRANTED` | Controller authorizes work |
| Any to Program | `IEF_BLOCKED` | Escalation to Controller |

---

## 3. Executable vs ACK-only Behavior Classification

### Executable Classifications

These trigger automated actions (dispatch, state mutation, artifact creation):

| Classification | Action |
|---------------|--------|
| `OPS_ACTION_REQUIRED` | Dispatch operator task |
| `OPS_EXECUTABLE_IDLE` | Trigger next queued task |
| `PROG_AUTHORIZATION_GRANTED` | Unlock blocked work items |
| `IEF_UNBLOCKED` | Resume paused workflows |
| `RUN_STAGE_READY` | Advance to next execution stage |
| `ADP_SYNC_TRIGGERED` | Execute external sync |

### ACK-only / Status Classifications

These represent state without triggering automated action:

| Classification | Meaning |
|---------------|---------|
| `OPS_WAITING_CODEX` | Awaiting external AI agent response |
| `IEF_WAITING_HUMAN` | Awaiting human decision |
| `OPS_WAITING_AGENT` | Awaiting agent availability |
| `KNW_REVIEW_RETURNED` | Review completed, no auto-action |
| `OPS_NO_ACTION_REQUIRED` | Cycle completed, no further work |
| `OPS_NO_SAFE_ACTION` | Safety check prevented action |
| `OPS_READY_FOR_PROGRAM_REVIEW` | Awaiting Controller review |
| `OPS_READY_FOR_HUMAN_SIGNOFF` | Awaiting human approval |
| `IEF_BLOCKED` | Work paused, requires intervention |

### Behavioral Contract

- **Executable** classifications MUST have: a defined handler, a timeout policy, and a failure escalation path.
- **ACK-only** classifications MUST have: a defined consumer, an expiration policy (or explicit no-expiration), and a resolution path.
- Any classification without a registered handler is treated as ACK-only by default.

---

## 4. Dispatch Authorization Semantics

### Authority Levels

| Level | Actor | Scope |
|-------|-------|-------|
| L0 | Human Owner | All classifications, all domains, override/revoke any |
| L1 | Program Controller | All `PROG_`, `IEF_` classifications; delegate `OPS_` and `RUN_` |
| L2 | PM Agent | `OPS_` classifications within authorized scope |
| L3 | Operator / Runner Agent | `RUN_` classifications for own tasks only |
| L4 | External Adapter | `ADP_` classifications for registered integrations |

### Delegation Chain

- L1 may delegate specific `OPS_` classifications to L2 with explicit scope (issue/PR/repo).
- L2 may delegate specific `RUN_` classifications to L3 with explicit task binding.
- Delegation is non-transitive: L3 cannot further delegate.

### Issuance Rules

| Classification | Authorized Issuers |
|---------------|-------------------|
| `PROG_AUTHORIZATION_GRANTED` | L0, L1 only |
| `IEF_BLOCKED` | Any level |
| `IEF_UNBLOCKED` | L0, L1, or issuer of corresponding BLOCKED |
| `OPS_ACTION_REQUIRED` | L1, L2 |
| `OPS_WAITING_HUMAN` | L2, L3 (escalation) |
| `RUN_EXECUTABLE_IDLE` | L3 only |
| `KNW_REVIEW_RETURNED` | L1, L2, designated reviewer |

### Revocation and Override

- L0 can revoke any classification at any time.
- L1 can revoke any classification below L1 authority.
- A classification can only be overridden by an equal or higher authority level.
- Revocation propagates to all consumers that received the original classification.

---

## 5. Dedupe Rules per Classification

### Dedupe Strategy Types

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| `IGNORE` | Duplicate is silently dropped | Status classifications (`WAITING_*`) |
| `UPDATE` | Duplicate replaces previous with new timestamp/context | Progress updates (`ACTION_REQUIRED` with new payload) |
| `ERROR` | Duplicate is rejected and logged | One-shot authorizations (`AUTHORIZATION_GRANTED`) |
| `COALESCE` | Multiple duplicates within a time window are merged | Bulk dispatches |

### Per-Classification Dedupe

| Classification | Strategy | Window | Rationale |
|---------------|----------|--------|-----------|
| `OPS_ACTION_REQUIRED` | `UPDATE` | — | New dispatch context replaces stale |
| `OPS_WAITING_CODEX` | `IGNORE` | — | Already waiting, no re-wait |
| `IEF_BLOCKED` | `IGNORE` | — | Already blocked |
| `IEF_UNBLOCKED` | `ERROR` | — | Must not double-unblock |
| `PROG_AUTHORIZATION_GRANTED` | `ERROR` | — | One-shot, replay unsafe |
| `RUN_EXECUTABLE_IDLE` | `IGNORE` | 300s | Prevent idle-loop re-entry |
| `OPS_READY_FOR_PROGRAM_REVIEW` | `UPDATE` | — | Latest review request wins |

### Anti-Loop Safeguards

- A classification that triggers an executable action MUST include a `dedupe_key` derived from the triggering context (comment_id, task_id, or decision_hash).
- If the same `dedupe_key` is seen within the classification dedupe window, the action is suppressed.
- After any state change (PR merge, new decision, stage advance), the dedupe key MUST be invalidated to allow re-processing of genuinely new events.
- The known failure mode from `g8_1_dispatched` (25 cycles of dead spinning) was caused by a stale dedupe key surviving a state transition. The registry must enforce explicit dedupe key invalidation on state changes.

---

## 6. Target Resolution Rules

### Classification to Target Mapping

| Classification Type | Resolution Target |
|-------------------|------------------|
| `OPS_ACTION_REQUIRED` | Target issue/PR specified in dispatch payload |
| `IEF_BLOCKED` | Issue where block was detected |
| `KNW_REVIEW_RETURNED` | PR that was reviewed |
| `PROG_AUTHORIZATION_GRANTED` | Issue/PR specified in authorization |
| `IEF_STAGE_READY` | Program-level (no specific issue) |

### Cross-Repo Classification Propagation

- Classifications issued in IEF-Program may propagate to IEF-Operations, IEF-Runners, IEF-Adapters, or IEF-Knowledge via the Coordinator relay mechanism.
- Propagated classifications retain their original `source_comment_id` and `dedupe_key` but gain a `propagated_to` field listing target repos.
- The receiving agent MUST validate the classification against its local registry subset before acting.

### Orphan Classification Handling

An orphan classification is one whose target issue/PR has been closed, merged, or deleted.

- **Executable orphans**: Logged as warning; action suppressed; classification auto-expires after 24h.
- **ACK-only orphans**: Logged as info; classification remains until explicit resolution or TTL expiry.
- Orphan detection runs as part of the mainline cron cycle.

---

## 7. Unsupported Classification Behavior

### Encountering Unknown Classifications

When an agent encounters a classification not in the registry:

1. **Log** the classification name, source, and context at WARN level.
2. **Do NOT execute** any action associated with the unknown classification.
3. **Report** to Coordinator as `PREFLIGHT_VOCABULARY_MISMATCH`.
4. Coordinator relays to Controller for resolution.

### Graceful Degradation vs Strict Rejection

| Context | Behavior |
|---------|----------|
| PM SKILL.md classification handler | Graceful: treat as ACK-only, log, continue cycle |
| Heartbeat decision processing | Strict: HOLD relay, report to Controller |
| Cross-repo propagation | Strict: reject propagation, log mismatch |
| Human-issued classification | Graceful: accept but flag for registry update |

### Alerting

- Repeated unknown classifications (more than 3 in 24h from same source) trigger escalation to Human Owner.
- A single unknown classification from a high-authority source (L0/L1) triggers immediate Controller notification.

---

## 8. Versioning and Migration Strategy

### Schema Versioning

- The registry schema uses semantic versioning: `MAJOR.MINOR.PATCH`.
- `MAJOR`: Breaking changes (removed classifications, changed behavior types).
- `MINOR`: Additive changes (new classifications, new fields).
- `PATCH`: Documentation clarifications, typo fixes.

### Current Version

The initial registry is version `0.1.0` — draft, non-enforcing, additive only.

### Backward Compatibility

- Classifications in use at the time of registry publication are grandfathered under their current names.
- Renamed classifications maintain a `deprecated_names` list for at least 2 minor versions.
- Removed classifications enter a `DEPRECATED` state for at least 1 major version before deletion.

### Migration Paths

| Scenario | Path |
|----------|------|
| Classification rename | Old name added to `deprecated_names`; new name becomes canonical; agents accept both during deprecation window |
| Classification removal | Status set to `DEPRECATED`; agents log warning on use; removed in next MAJOR version |
| Classification split (1 to N) | Original marked `SPLIT`; new entries reference `supersedes: [original]`; original deprecated after all consumers migrate |
| Classification merge (N to 1) | All originals marked `MERGED_INTO`; new entry references `supersedes: [list]` |

---

## 9. Relationship to PM SKILL.md and AGENTS.md

### Current State

PM SKILL.md contains the authoritative classification list at approximately lines 926-1001 (75 entries). AGENTS.md contains operational notes about classification behavior (dedupe, preflight, escalation).

### Registry Connection

The registry does NOT replace PM SKILL.md entries. Instead:

- PM SKILL.md remains the **runtime reference** for the PM agent classification handlers.
- The registry provides the **canonical schema** — naming rules, dedupe policies, authorization levels — that PM SKILL.md entries should conform to.
- Over time, PM SKILL.md may reference the registry via `see: CLASSIFICATION_REGISTRY#OPS_WAITING_CODEX` rather than duplicating definitions.

### Agent-Specific Consumption Patterns

| Agent | Consumes From | Validation |
|-------|--------------|------------|
| PM | PM SKILL.md (primary), Registry (reference) | Preflight validation against registry on heartbeat |
| Coordinator | AGENTS.md + Registry | Validates Controller decisions against registry |
| Operator | Dispatch payload only | Trusts dispatcher validation |
| Dashboard | Registry (full read) | No validation needed |

### SYNC_FROM_GITHUB Integration

- When PM syncs decisions from GitHub (`SYNC_FROM_GITHUB`), each decision classification label is validated against the registry.
- Unrecognized labels are held (not consumed) and reported as `PREFLIGHT_VOCABULARY_MISMATCH`.
- This prevents the known failure mode where an unrecognized Controller classification causes PM to spin without progress.

---

## 10. Proposed Registry Location and Schema

### Location

```
docs/governance/CLASSIFICATION_REGISTRY.yaml
```

YAML is chosen for human readability, machine parseability, and compatibility with Spectral lint rules (proposed in D7 Spec Governance Briefing).

### Schema Structure

```yaml
schema_version: "0.1.0"
classifications:
  - name: string                    # UPPER_SNAKE_CASE canonical name
    domain: string                  # PROG | OPS | RUN | ADP | KNW | IEF
    stage: string[]                 # Applicable IEF stages
    behavior_type: string           # EXECUTABLE | ACK_ONLY
    auth_level: integer             # 0-4 (minimum authority to issue)
    authorized_issuers: string[]    # Explicit issuer list
    dedupe_strategy: string         # IGNORE | UPDATE | ERROR | COALESCE
    dedupe_window_seconds: integer  # 0 = no window
    ttl_seconds: integer            # 0 = no TTL
    description: string             # Human-readable description
    deprecated_names: string[]      # Previous names (migration support)
    status: string                  # ACTIVE | DEPRECATED | SPLIT | MERGED_INTO
    supersedes: string[]            # For split/merge migrations
    see: string                     # Cross-reference (e.g., PM SKILL.md line)
```

### Example Entries

```yaml
schema_version: "0.1.0"
classifications:
  - name: OPS_ACTION_REQUIRED
    domain: OPS
    stage: [operations]
    behavior_type: EXECUTABLE
    auth_level: 1
    authorized_issuers: [controller, pm]
    dedupe_strategy: UPDATE
    dedupe_window_seconds: 0
    ttl_seconds: 86400
    description: "Dispatch operator task with specified payload"
    deprecated_names: []
    status: ACTIVE
    supersedes: []
    see: "PM SKILL.md L930"

  - name: OPS_WAITING_CODEX
    domain: OPS
    stage: [operations]
    behavior_type: ACK_ONLY
    auth_level: 2
    authorized_issuers: [pm, operator]
    dedupe_strategy: IGNORE
    dedupe_window_seconds: 0
    ttl_seconds: 0
    description: "Awaiting external AI agent (Codex) response"
    deprecated_names: []
    status: ACTIVE
    supersedes: []
    see: "PM SKILL.md L935"

  - name: IEF_BLOCKED
    domain: IEF
    stage: [program, operations, runners, adapters, knowledge]
    behavior_type: ACK_ONLY
    auth_level: 4
    authorized_issuers: [any]
    dedupe_strategy: IGNORE
    dedupe_window_seconds: 0
    ttl_seconds: 0
    description: "Work paused; requires intervention from higher authority"
    deprecated_names: []
    status: ACTIVE
    supersedes: []
    see: "AGENTS.md Controller section"

  - name: PROG_AUTHORIZATION_GRANTED
    domain: PROG
    stage: [program]
    behavior_type: EXECUTABLE
    auth_level: 1
    authorized_issuers: [controller, human_owner]
    dedupe_strategy: ERROR
    dedupe_window_seconds: 0
    ttl_seconds: 604800
    description: "Controller authorizes specific scope of work"
    deprecated_names: []
    status: ACTIVE
    supersedes: []
    see: "AGENTS.md Controller decisions"

  - name: IEF_STAGE_READY
    domain: IEF
    stage: [program, operations]
    behavior_type: EXECUTABLE
    auth_level: 1
    authorized_issuers: [controller]
    dedupe_strategy: UPDATE
    dedupe_window_seconds: 3600
    ttl_seconds: 0
    description: "Signals that a lifecycle stage is ready for transition"
    deprecated_names: []
    status: ACTIVE
    supersedes: []
    see: "AGENTS.md Stage transitions"

  - name: KNW_REVIEW_RETURNED
    domain: KNW
    stage: [knowledge]
    behavior_type: ACK_ONLY
    auth_level: 1
    authorized_issuers: [controller, pm, reviewer]
    dedupe_strategy: UPDATE
    dedupe_window_seconds: 0
    ttl_seconds: 0
    description: "Spec/document review completed"
    deprecated_names: []
    status: ACTIVE
    supersedes: []
    see: "PM SKILL.md L945"

  - name: OPS_NO_ACTION_REQUIRED
    domain: OPS
    stage: [operations]
    behavior_type: ACK_ONLY
    auth_level: 2
    authorized_issuers: [pm]
    dedupe_strategy: IGNORE
    dedupe_window_seconds: 0
    ttl_seconds: 0
    description: "PM cycle completed with no actionable work"
    deprecated_names: []
    status: ACTIVE
    supersedes: []
    see: "PM SKILL.md L950"

  - name: OPS_NO_SAFE_ACTION
    domain: OPS
    stage: [operations]
    behavior_type: ACK_ONLY
    auth_level: 2
    authorized_issuers: [pm, operator]
    dedupe_strategy: IGNORE
    dedupe_window_seconds: 0
    ttl_seconds: 0
    description: "Safety check prevented action execution"
    deprecated_names: []
    status: ACTIVE
    supersedes: []
    see: "PM SKILL.md L955"
```

---

## 11. Migration Plan from PM SKILL.md Appendix

### Phase 1: Registry Creation (Current — v0.1.0)

- Create the registry YAML with the 13 known active classifications listed in this plan context section.
- PM SKILL.md remains authoritative; registry is reference-only.
- No enforcement; no behavior changes.

### Phase 2: Cross-Reference (v0.2.0)

- Add `see:` cross-references from registry entries to PM SKILL.md line numbers.
- Add `registry_ref:` annotations to PM SKILL.md entries pointing back to registry.
- PM heartbeat begins logging WARN when it encounters a classification not in the registry (non-blocking).

### Phase 3: Preflight Enforcement (v0.3.0)

- PM heartbeat performs preflight validation: unrecognized classifications are held and reported.
- Registry becomes the validation source of truth for SYNC_FROM_GITHUB.
- PM SKILL.md handlers remain unchanged but are expected to align with registry definitions.

### Phase 4: Full Extraction (v1.0.0)

- Extract all 75+ classifications from PM SKILL.md into the registry.
- PM SKILL.md references registry entries instead of duplicating definitions.
- PM SKILL.md retains handler logic but classification metadata lives in the registry.
- Spectral lint rules validate registry conformance in CI.

### Deprecation Timeline

| Milestone | Target | Gate |
|-----------|--------|------|
| Registry draft published | v0.1.0 | This document |
| Cross-references added | v0.2.0 | Controller approval |
| Preflight enforcement active | v0.3.0 | Controller + Human approval |
| Full extraction complete | v1.0.0 | Controller + Human + CI validation |

### Validation Approach

- **Phase 1-2**: Manual spot-check that registry entries match PM SKILL.md definitions.
- **Phase 3**: Automated diff between registry names and PM SKILL.md classification list; discrepancies reported but not blocking.
- **Phase 4**: CI job validates that every classification in PM SKILL.md has a corresponding ACTIVE registry entry with matching metadata.

---

## Appendix: Current Active Classifications (Inventory)

These are the classifications currently observed across IEF-Program, IEF-Operations, and related repos:

| Name | Observed Domain | Behavior (Inferred) |
|------|----------------|-------------------|
| WAITING_CODEX | OPS | ACK_ONLY |
| WAITING_HUMAN | IEF | ACK_ONLY |
| BLOCKED | IEF | ACK_ONLY |
| UNBLOCKED | IEF | EXECUTABLE |
| WAITING_AGENT | OPS | ACK_ONLY |
| REVIEW_RETURNED | KNW | ACK_ONLY |
| EXECUTABLE_IDLE | RUN | EXECUTABLE |
| STAGE_READY | IEF | EXECUTABLE |
| ACTION_REQUIRED | OPS | EXECUTABLE |
| NO_ACTION_REQUIRED | OPS | ACK_ONLY |
| NO_SAFE_ACTION | OPS | ACK_ONLY |
| READY_FOR_PROGRAM_REVIEW | OPS | ACK_ONLY |
| READY_FOR_HUMAN_SIGNOFF | OPS | ACK_ONLY |

**Note:** This inventory is intentionally non-exhaustive. The 75+ classifications in PM SKILL.md are not migrated here per the hard constraints of this directive. Full inventory is deferred to Phase 4.
