# System Hardening Plan

**Status:** Planning Document — No Runtime Changes  
**Created:** 2026-06-28  
**Authorization:** Controller-directed via Coordinator relay (SYSTEM_HARDENING_PLAN_AUTHORIZED)  
**Source:** IEF-Program#16 comment 4824590867  
**Baseline:** IEF-Program `main` @ current  
**Companion docs:** `CLASSIFICATION_REGISTRY_SPEC_PLAN.md`, IEF-Operations `ENFORCEMENT_ACTIVATION_POLICY.md`, IEF-Operations `EVOLUTION_GUARDRAIL_SPEC.md`, IEF-Operations `VERIFICATION_HARDENING_SPEC.md`, IEF-Operations `FAILURE_REPLAY_MODEL.md`

---

## ⚠️ Scope Statement

**This document is a planning artifact only. No runtime behavior is changed by this PR.**

- No registry runtime enforcement is implemented
- No PM classification handler is modified
- No Spectral CI checks are activated
- No build-breaking checks are introduced
- No runtime state WAL is implemented
- No changes to existing dashboard, PM, or Operator behavior

All hardening streams defined here require separate Controller authorization for each phase transition.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Hardening Phase Model](#2-hardening-phase-model)
3. [Stream A: Classification Registry Runtime Behavior](#3-stream-a-classification-registry-runtime-behavior)
4. [Stream B: Spectral CI Activation Path](#4-stream-b-spectral-ci-activation-path)
5. [Stream C: State Consistency Model](#5-stream-c-state-consistency-model)
6. [Stream D: Operational Hardening](#6-stream-d-operational-hardening)
7. [Cross-Stream Dependencies](#7-cross-stream-dependencies)
8. [Risk Assessment](#8-risk-assessment)
9. [Activation Criteria Per Stream](#9-activation-criteria-per-stream)
10. [Compliance and Audit Trail](#10-compliance-and-audit-trail)

---

## 1. Overview

### 1.1 Motivation

The IEF program has reached Stage G (Governance Hardening) with multiple governance specifications delivered and merged:

- **Classification Registry Spec Plan** (IEF-Program) — naming rules, dedupe policies, authorization levels
- **Execution Enforcement Phase Model** (IEF-Operations, IEF-Program) — 5-phase execution gating
- **Enforcement Activation Policy** (IEF-Operations) — phase model activation rules
- **Evolution Guardrail Spec** (IEF-Operations) — self-evolution boundaries
- **Verification Hardening Spec** (IEF-Operations) — verification pipeline requirements
- **Failure Replay Model** (IEF-Operations) — failure recovery and replay

These specifications are currently **declarative only** — they define what the system should do, but do not implement runtime enforcement. This plan defines the path from declarative governance to enforced governance across four hardening streams.

### 1.2 Design Principles

1. **Progressive enforcement**: Move from invisible (planning) → visible (report-only) → advisory (warning) → mandatory (enforcement). Each phase is gated.
2. **No surprise breakage**: No CI check goes from nonexistent to blocking without passing through report-only and warning first.
3. **Explicit activation**: Each stream activates independently. Controller must authorize each phase transition.
4. **Rollback safety**: Every phase must be reversible. If warning-phase produces excessive noise, it can be reverted to report-only without system changes.
5. **Observability first**: Before enforcing anything, the system must be able to report on what it would enforce.

### 1.3 Hardening Streams

| Stream | Scope | Current State | Target State |
|--------|-------|--------------|--------------|
| A: Classification Registry | Vocabulary validation, unknown classification handling | Planning only (registry spec published) | Runtime enforcement with preflight checks |
| B: Spectral CI | Spec governance lint rules, schema validation | No CI checks exist | Required CI check on all governed artifacts |
| C: State Consistency | State file integrity, crash recovery, dedupe/replay | Repair-first (ad hoc) | WAL-based transaction model |
| D: Operational Hardening | PM/Operator behavioral hardening, escalation rules | Implicit (agent-level) | Explicit policy with monitoring |

---

## 2. Hardening Phase Model

All four streams follow the same four-phase progression:

```
PLANNING ──► REPORT-ONLY ──► WARNING ──► ENFORCEMENT
   (0)           (1)           (2)          (3)
```

### 2.1 Phase Definitions

#### Phase 0: Planning (Current)

- **What it means**: The hardening requirement is documented. No runtime components exist.
- **Runtime impact**: None.
- **Exit criteria**: Specification is published, reviewed, and Controller-approved.

#### Phase 1: Report-Only

- **What it means**: The system collects data about what *would* be enforced and reports it, but takes no action.
- **Runtime impact**: Additional logging/metrics only. No behavior changes.
- **Exit criteria**: Report data shows stable baseline; false-positive rate is acceptable; Controller reviews report output.

#### Phase 2: Warning

- **What it means**: The system actively warns when a violation is detected, but does not block or fail.
- **Runtime impact**: Warnings appear in logs, comments, or dashboard. CI passes with warnings.
- **Exit criteria**: Warning rate is below threshold; all known violations are tracked; Controller approves enforcement transition.

#### Phase 3: Enforcement

- **What it means**: The system actively prevents violations. CI fails on violations. Dispatches are blocked on preflight failures.
- **Runtime impact**: Build-breaking checks, dispatch holds, classification rejection.
- **Exit criteria**: System operates stably under enforcement for a defined soak period.

### 2.2 Phase Transition Rules

| Transition | Requires | Reversible |
|-----------|----------|-----------|
| Planning → Report-Only | Controller authorization, spec review | Yes (remove reporting) |
| Report-Only → Warning | Controller authorization, report baseline review | Yes (revert to report-only) |
| Warning → Enforcement | Controller + Human authorization, warning soak period | Yes (revert to warning) |
| Any → Planning (rollback) | Controller authorization | N/A (terminal rollback) |

### 2.3 Per-Stream Phase Independence

Each stream advances through phases independently. It is valid for Stream A to be in Warning while Stream B is still in Report-Only. Cross-stream dependencies are documented in Section 7.

---

## 3. Stream A: Classification Registry Runtime Behavior

### 3.1 Current State

The Classification Registry Spec Plan (`CLASSIFICATION_REGISTRY_SPEC_PLAN.md`) defines:
- Naming rules (UPPER_SNAKE_CASE, namespace prefixes)
- Behavior types (EXECUTABLE vs ACK-only)
- Dedupe policies per classification
- Authorization levels (L0-L4)
- Versioning and migration strategy

However, the registry is **reference only**. No runtime component validates classifications against it.

### 3.2 Runtime Behavior Design

#### 3.2.1 Registry Loading

The registry should be loaded as a static lookup table at agent startup:

```
Registry Source: docs/governance/CLASSIFICATION_REGISTRY.yaml
Load Strategy: Parse YAML at agent init, cache in memory
Fallback: If registry is missing or unparseable, log WARN and operate in permissive mode (treat all classifications as ACK-only)
```

**Constraints**:
- Registry is loaded once per agent lifecycle (no hot-reload)
- Registry version is logged at startup for audit
- Registry file path is configurable but defaults to the canonical location

#### 3.2.2 Unknown Classification Handling

When an agent encounters a classification not in the registry:

| Context | Phase 1 (Report-Only) | Phase 2 (Warning) | Phase 3 (Enforcement) |
|---------|----------------------|-------------------|----------------------|
| PM heartbeat (SYNC_FROM_GITHUB) | Log WARN, consume as ACK-only | Log WARN, hold for 1 cycle, then consume as ACK-only | Reject, do not consume, post PREFLIGHT_VOCABULARY_MISMATCH |
| Coordinator relay processing | Log WARN, process normally | Log WARN, flag in relay summary | Reject relay, post PREFLIGHT_VOCABULARY_MISMATCH |
| Operator dispatch payload | Log WARN, proceed | Log WARN, proceed with flag | Block dispatch, require valid classification |
| Cross-repo propagation | Log WARN, propagate | Log WARN, propagate with flag | Block propagation |

#### 3.2.3 PM / Coordinator / Controller Vocabulary Compatibility

**Problem**: The Controller (ChatGPT), Coordinator, and PM may use different names for the same classification (e.g., `ACTION_REQUIRED` vs `OPS_ACTION_REQUIRED`).

**Design**:
- The registry defines canonical names with `deprecated_names` for aliases
- During Phase 1-2: Agent attempts fuzzy matching against `deprecated_names` and logs mismatches
- During Phase 3: Agent requires exact canonical name match; aliases are rejected with a suggestion

**Compatibility table** (initial):

| Observed Name | Canonical Name | Domain |
|---------------|---------------|--------|
| `ACTION_REQUIRED` | `OPS_ACTION_REQUIRED` | OPS |
| `WAITING_CODEX` | `OPS_WAITING_CODEX` | OPS |
| `BLOCKED` | `IEF_BLOCKED` | IEF |
| `AUTHORIZATION_GRANTED` | `PROG_AUTHORIZATION_GRANTED` | PROG |
| `STAGE_READY` | `IEF_STAGE_READY` | IEF |

#### 3.2.4 Migration from SKILL.md Appendix

Per the Classification Registry Spec Plan §11:

| Phase | Registry Version | SKILL.md Relationship |
|-------|-----------------|----------------------|
| Phase 1 (current) | v0.1.0 | Registry is reference; SKILL.md is authoritative |
| Phase 2 | v0.2.0 | Cross-references added bidirectionally |
| Phase 3 | v0.3.0 | Preflight validation against registry |
| Phase 4 | v1.0.0 | Full extraction; SKILL.md references registry |

#### 3.2.5 Versioning Model

- Registry uses semantic versioning (`MAJOR.MINOR.PATCH`)
- Agents log the registry version they loaded
- Preflight checks include version compatibility (agent requires registry ≥ minimum version)
- Registry version is included in delivery reports and audit trails

#### 3.2.6 ACK-only vs Executable Classification Runtime

| Behavior Type | Runtime Effect | Timeout | Escalation |
|--------------|---------------|---------|-----------|
| EXECUTABLE | Triggers defined handler | Must have timeout policy | Escalate on timeout |
| ACK_ONLY | Logged, state recorded | Optional TTL | Escalate on TTL expiry |
| Unknown (permissive mode) | Treated as ACK_ONLY | No timeout | Reported as mismatch |

### 3.3 Stream A Phase Plan

| Phase | Deliverable | Prerequisites | Activation Criteria |
|-------|------------|---------------|-------------------|
| **0: Planning** | This plan + Registry Spec Plan | ✅ Complete | Controller approval of plan |
| **1: Report-Only** | Registry YAML published; agents log unknown classifications | Registry v0.1.0 published | Controller authorizes report-only activation |
| **2: Warning** | Agents post warnings for unknown classifications; preflight soft-check | 2+ weeks of report-only data; false-positive rate < 5% | Controller authorizes warning activation |
| **3: Enforcement** | Agents reject unknown classifications; preflight hard-check blocks dispatches | 2+ weeks of warning data; all known mismatches resolved | Controller + Human authorize enforcement |

### 3.4 Risk Assessment — Stream A

| Risk | Severity | Mitigation |
|------|---------|-----------|
| False positives block valid dispatches | High | Graduated phases; permissive fallback |
| Registry out of sync with PM SKILL.md | Medium | Cross-reference phase (v0.2.0) before enforcement |
| Controller uses non-canonical names | Medium | Alias table; suggestion on rejection |
| Registry file corrupted or deleted | Low | Fallback to permissive mode; CI validation (Stream B) |

---

## 4. Stream B: Spectral CI Activation Path

### 4.1 Current State

No Spectral CI checks exist for governed artifacts. Governance documents (specs, policies, contracts) are reviewed manually. Schema validation is ad hoc.

Companion specs that define lintable rules:
- `EVOLUTION_GUARDRAIL_SPEC.md` — evolution boundaries
- `SPEC_GOVERNANCE_SPECTRAL_LINT_PLAN.md` — lint scoping plan
- `CLASSIFICATION_REGISTRY_SPEC_PLAN.md` — classification naming rules
- `VERIFICATION_HARDENING_SPEC.md` — verification pipeline requirements

### 4.2 Target State

Spectral CI checks that validate governed artifacts against governance rules, progressing from report-only to required check.

### 4.3 Ruleset Baseline

#### 4.3.1 Governance Document Rules

| Rule ID | Description | Severity | Source |
|---------|------------|----------|--------|
| `IEF-GOV-001` | Governance doc must have Status header | WARN | Evolution Guardrail §2.2 |
| `IEF-GOV-002` | Governance doc must have Authority/Authorization field | WARN | Evolution Guardrail §2.2 |
| `IEF-GOV-003` | Governance doc must have Created date | WARN | Evolution Guardrail §2.2 |
| `IEF-GOV-004` | Governance doc must have Baseline SHA reference | WARN | Evolution Guardrail §2.2 |
| `IEF-GOV-005` | Governance doc must reference companion specs where applicable | INFO | Evolution Guardrail §1 |
| `IEF-GOV-006` | Governance doc must not contain executable code (planning docs) | ERROR | Evolution Guardrail §3 |
| `IEF-GOV-007` | Frozen spec must not be modified without Controller authorization | ERROR | Evolution Guardrail §3 |

#### 4.3.2 Schema Rules

| Rule ID | Description | Severity | Source |
|---------|------------|----------|--------|
| `IEF-SCHEMA-001` | Schema must declare version (semver) | WARN | Spec Governance Lint Plan |
| `IEF-SCHEMA-002` | Schema must not remove required fields (breaking change) | ERROR | Evolution Guardrail §3.1 |
| `IEF-SCHEMA-003` | Schema must use JSON Schema Draft 2020-12 | WARN | Spec Governance Lint Plan |
| `IEF-SCHEMA-004` | Schema additions must be backward-compatible | WARN | Evolution Guardrail §2.1 |

#### 4.3.3 Classification Rules

| Rule ID | Description | Severity | Source |
|---------|------------|----------|--------|
| `IEF-CLASS-001` | Classification name must be UPPER_SNAKE_CASE | WARN | Registry Spec Plan §1 |
| `IEF-CLASS-002` | Classification must use registered namespace prefix | WARN | Registry Spec Plan §1 |
| `IEF-CLASS-003` | Classification must not exceed 64 characters | INFO | Registry Spec Plan §1 |
| `IEF-CLASS-004` | Classification must have behavior_type declared | WARN | Registry Spec Plan §3 |

### 4.4 Target Repos and File Globs

| Repo | File Globs | Ruleset |
|------|-----------|---------|
| `everwork-ai/IEF-Operations` | `docs/governance/*.md`, `docs/governance/*.yaml` | GOV, SCHEMA, CLASS |
| `everwork-ai/IEF-Program` | `docs/governance/*.md`, `docs/governance/*.yaml` | GOV, SCHEMA, CLASS |
| `everwork-ai/IEF-Runners` | `docs/contracts/*.md`, `docs/contracts/*.yaml` | GOV, SCHEMA |
| `everwork-ai/IEF-Knowledge` | `docs/contracts/*.md`, `docs/contracts/*.yaml` | GOV, SCHEMA |
| `everwork-ai/IEF-Sandbox` | (none — sandbox exempt) | — |

### 4.5 Cleanup Prerequisites

Before Spectral CI can move from Report-Only to Warning:

1. **Existing docs audit**: All existing governance docs must be audited against the ruleset. Known violations must be documented.
2. **Baseline report**: A baseline report showing current violation count must be published.
3. **Fix backlog**: Critical violations (ERROR severity) in existing docs must be fixed or explicitly grandfathered.
4. **Ruleset stabilization**: Ruleset must be stable for at least 2 weeks without changes.

### 4.6 Rollout Sequencing

| Phase | Scope | PR Check Behavior | Prerequisites |
|-------|-------|------------------|---------------|
| **0: Planning** | This plan | None | ✅ Complete |
| **1: Report-Only** | IEF-Operations only | Spectral runs in CI but does not report status | Ruleset finalized; Spectral config published |
| **1b: Report-Only Expand** | All governed repos | Spectral runs in CI but does not report status | IEF-Operations report-only stable for 1+ week |
| **2: Warning** | All governed repos | Spectral reports warnings as PR check (non-blocking) | Baseline audit complete; violation count < threshold |
| **3: Enforcement** | All governed repos | Spectral is required check; PRs with ERROR violations cannot merge | All ERROR violations resolved or grandfathered; Controller + Human approval |

### 4.7 Stream B Phase Plan

| Phase | Deliverable | Prerequisites | Activation Criteria |
|-------|------------|---------------|-------------------|
| **0: Planning** | This plan | ✅ Complete | Controller approval |
| **1: Report-Only** | `.spectral.yaml` config; CI workflow running in report mode | Ruleset finalized | Controller authorizes CI activation (report-only) |
| **2: Warning** | CI check posts warnings on PRs | Baseline audit; violation count documented | Controller authorizes warning mode |
| **3: Enforcement** | CI check is required; blocks merge on ERROR | All ERROR violations resolved; 2+ weeks warning soak | Controller + Human authorize enforcement |

### 4.8 Risk Assessment — Stream B

| Risk | Severity | Mitigation |
|------|---------|-----------|
| CI blocks valid PRs due to false-positive rules | High | Report-only phase validates rules against real PRs |
| Ruleset churn creates moving target | Medium | Ruleset stabilization period before warning phase |
| Existing docs have too many violations to fix | Medium | Grandfathering policy; fix backlog before enforcement |
| Spectral configuration drift across repos | Low | Centralized Spectral config in IEF-Operations; other repos reference it |

---

## 5. Stream C: State Consistency Model

### 5.1 Current State

The IEF system uses several state files that are critical to correct operation:

| State File | Location | Purpose |
|-----------|----------|---------|
| `last_dispatch.json` | `IEF-Orchestration/state/` | PM dispatch ledger, dedupe keys, cycle tracking |
| `tasks.json` | `IEF-Operations/` | Task registry, phase tracking |
| Trigger JSONs | `IEF-Orchestration/triggers/` | Operator trigger payloads |
| Delivery reports | `IEF-Orchestration/reports/` | Operator delivery evidence |

**Current model**: Repair-first. When state inconsistency is detected, agents attempt to repair it (re-fetch from GitHub, re-compute from PR state). There is no formal consistency model, no transaction boundaries, and no crash recovery protocol.

**Known failure modes**:
1. **Partial write**: Agent writes to `last_dispatch.json` but crashes before git push. Local and remote state diverge.
2. **Stale dedupe key**: Dedupe key survives a state transition, causing the agent to skip a genuinely new event (observed: `g8_1_dispatched` dead spin, 25 cycles).
3. **Concurrent modification**: Two agents (PM and Operator) modify overlapping state files simultaneously.
4. **Lost delivery report**: Operator completes work but crashes before posting delivery report. Work is duplicated on next cycle.

### 5.2 State File Consistency Rules

#### 5.2.1 Invariants

| Invariant | Scope | Detection |
|-----------|-------|-----------|
| **Single writer**: Only one agent writes to a state file at a time | All state files | File lock / git push conflict |
| **Monotonic head SHA**: Head SHA in state must be ≥ last known head SHA | `last_dispatch.json` | Compare with GitHub API |
| **Dedupe key freshness**: Dedupe keys must be invalidated on state transitions | `last_dispatch.json` | Explicit invalidation on merge/dispatch/decision |
| **Delivery report existence**: Every completed dispatch must have a delivery report | `last_dispatch.json` ↔ `reports/` | Cross-reference check |
| **Trigger JSON atomicity**: Trigger JSON must be fully written before Operator reads it | `triggers/` | Write-to-temp-then-rename pattern |

#### 5.2.2 Consistency Levels

| Level | Description | Use Case |
|-------|------------|---------|
| **Eventual** | State will converge eventually; temporary inconsistency is acceptable | Dashboard data, metrics |
| **Strong** | State must be consistent before next operation proceeds | Dispatch ledger, delivery reports |
| **Transactional** | Multi-step operation either fully completes or fully rolls back | Dispatch + commit + push + report |

### 5.3 WAL / Transaction Model Options

#### Option A: Git-Based WAL (Recommended for Phase 1)

Use git itself as a write-ahead log:

```
1. Create a unique branch for the operation: `wal/<agent>/<operation_id>`
2. Write state changes to the branch
3. Commit with structured message: [WAL] <operation_id> <step>
4. Push to remote (WAL is now durable)
5. Merge WAL branch into target branch
6. Delete WAL branch (cleanup)
```

**Pros**: Uses existing infrastructure; crash recovery is branch-based; auditable.  
**Cons**: Branch proliferation; merge conflicts possible; slow for high-frequency operations.

#### Option B: Local WAL File (Recommended for Phase 2)

Append-only log file alongside state files:

```jsonl
{"ts":"2026-06-28T12:00:00Z","op":"BEGIN","id":"dispatch_001","agent":"pm"}
{"ts":"2026-06-28T12:00:01Z","op":"WRITE","file":"last_dispatch.json","sha":"abc123"}
{"ts":"2026-06-28T12:00:02Z","op":"PUSH","branch":"main","sha":"abc123"}
{"ts":"2026-06-28T12:00:03Z","op":"COMMIT","id":"dispatch_001"}
```

**Recovery protocol**: On startup, agent reads WAL. If last entry is `BEGIN` or `WRITE` without `COMMIT`, the operation is incomplete. Agent can either:
- Roll forward: Complete the remaining steps
- Roll back: Revert the state file to pre-operation state (using git)

**Pros**: Fast; no branch proliferation; explicit transaction boundaries.  
**Cons**: WAL file itself can be corrupted; requires discipline to always write WAL entries.

#### Option C: Database-Backed State (Future — Phase 3+)

Replace file-based state with a lightweight embedded database (SQLite):

**Pros**: Full ACID transactions; built-in crash recovery; queryable.  
**Cons**: Major architectural change; requires new tooling; breaks existing workflows.

**Assessment**: Option C is out of scope for this plan. It is noted as a future option if file-based state reaches its limits.

### 5.4 Crash Recovery

#### 5.4.1 Recovery Scenarios

| Crash Point | Detection | Recovery |
|------------|-----------|---------|
| After state file write, before git commit | WAL has WRITE but no COMMIT | Roll back: `git checkout -- <file>` |
| After git commit, before push | Local has commit not on remote | Roll forward: retry push |
| After push, before delivery report | PR has commit but no report comment | Roll forward: post delivery report |
| After delivery report, before dispatch update | Report exists but dispatch not updated | Roll forward: update dispatch ledger |
| Mid-write to state file | File is truncated or invalid JSON | Roll back: restore from git HEAD |

#### 5.4.2 Recovery Protocol

```
1. Agent starts up
2. Check for WAL entries (if WAL is implemented)
3. If incomplete transaction found:
   a. Determine crash point from WAL entries
   b. Apply recovery action (roll forward or roll back)
   c. Log recovery action
   d. Mark transaction as RECOVERED
4. If no WAL, use heuristic recovery:
   a. Compare local state with remote (GitHub API)
   b. If local is ahead: push
   c. If local is behind: pull
   d. If diverged: alert human
```

### 5.5 Dedupe / Replay Protection

#### 5.5.1 Dedupe Key Lifecycle

```
Creation → Active → Consumed → Invalidated
   │          │         │           │
   │          │         │           └─ State transition occurs; key is no longer valid
   │          │         └─ Agent processes the event associated with this key
   │          └─ Key is in the dedupe window; duplicates are suppressed
   └─ Key is generated from event context (comment_id, task_id, etc.)
```

#### 5.5.2 Dedupe Key Invalidation Triggers

| Event | Keys Invalidated |
|-------|-----------------|
| PR merged on target repo | All dispatch keys for that PR |
| New Controller decision posted | All relay keys for previous decisions on same issue |
| Phase transition (e.g., Stage D → Stage G) | All keys from previous phase |
| Active anchor change | All keys referencing old anchor |
| Manual Controller override | All keys in the affected scope |

#### 5.5.3 Replay Protection

- Every dispatched operation includes a `replay_guard` field: `{ dedupe_key, source_comment_id, head_sha_at_dispatch }`
- Before executing an operation, the agent checks:
  1. Is this `dedupe_key` already in the consumed set?
  2. Is the `source_comment_id` still valid (not superseded)?
  3. Is the `head_sha_at_dispatch` still the current head? (If not, re-validate.)
- If any check fails, the operation is skipped and logged.

### 5.6 Audit Trail Expectations

Every state mutation must produce an audit entry:

| Field | Description |
|-------|------------|
| Timestamp | ISO-8601 |
| Agent | Which agent made the change |
| Operation | What was done (write, push, dispatch, etc.) |
| File(s) | Which state files were affected |
| Before SHA | Git SHA before the change |
| After SHA | Git SHA after the change |
| Dedupe Key | Key associated with this operation |
| Transaction ID | WAL transaction ID (if applicable) |

Audit entries are append-only and stored in `IEF-Orchestration/logs/audit.jsonl`.

### 5.7 Stream C Phase Plan

| Phase | Deliverable | Prerequisites | Activation Criteria |
|-------|------------|---------------|-------------------|
| **0: Planning** | This plan; consistency rules documented | ✅ Complete | Controller approval |
| **1: Report-Only** | Audit trail logging; state consistency checks (non-blocking) | Audit log format defined | Controller authorizes audit logging activation |
| **2: Warning** | Consistency violations reported; dedupe key lifecycle tracking | 2+ weeks of audit data; common inconsistency patterns cataloged | Controller authorizes warning mode |
| **3: Enforcement** | WAL-based transactions; crash recovery protocol active; dedupe hard-checks | WAL implementation complete; recovery tested; Controller + Human approval | Controller + Human authorize enforcement |

### 5.8 Risk Assessment — Stream C

| Risk | Severity | Mitigation |
|------|---------|-----------|
| WAL implementation introduces new bugs | High | Extensive testing; Phase 1-2 observation period |
| Audit log grows unbounded | Medium | Log rotation policy; archival |
| Crash recovery makes wrong decision (roll forward vs back) | High | Conservative default (alert human); only auto-recover well-understood scenarios |
| Dedupe key invalidation misses a trigger | Medium | Comprehensive trigger list; periodic full-scan audit |
| Git-based WAL (Option A) creates branch proliferation | Low | Cleanup job; Option B for Phase 2 |

---

## 6. Stream D: Operational Hardening

### 6.1 Current State

PM and Operator behavior is governed by AGENTS.md, SKILL.md, and ad hoc operational notes. Several behavioral rules are implicit — agents follow them because they're in the prompt, not because they're monitored or enforced.

**Known operational pain points**:
1. **Push escalation cooldown**: No formal rule for how long to wait between L2 pushes.
2. **Duplicate directive suppression**: PM sometimes processes the same directive twice when dedupe keys are stale.
3. **Pending capability gap lifecycle**: Capability gaps are reported but not tracked to resolution.
4. **PM quiet-cycle behavior**: PM may spin in quiet cycles without making progress.
5. **Dashboard refresh failure reporting**: No standardized format for dashboard refresh failures.

### 6.2 Push Escalation Cooldown Rules

#### 6.2.1 Escalation Levels

| Level | Action | Cooldown | Max Repeats |
|-------|--------|----------|-------------|
| L1: Standard dispatch | Normal Operator dispatch | 1 cycle | Unlimited |
| L2: Hard push | Public comment requesting Controller decision | 24 hours | 3 |
| L3: Planning request | Formal planning request to Controller | 48 hours | 2 |
| L4: Human escalation | Direct notification to Human Owner | 72 hours | 1 |

#### 6.2.2 Cooldown Enforcement

- **Phase 1 (Report-Only)**: Log when cooldown is violated; no action taken.
- **Phase 2 (Warning)**: Post warning comment when cooldown is about to be violated (e.g., "L2 push was posted 18h ago; cooldown is 24h; suppressing this cycle").
- **Phase 3 (Enforcement)**: Hard block: agent cannot post escalation comment until cooldown has elapsed.

### 6.3 Duplicate Directive Suppression

#### 6.3.1 Detection Rules

A directive is considered duplicate if:

1. Same `source_comment_id` as a previously processed directive, OR
2. Same `dedupe_key` as a previously processed directive AND the state has not changed since, OR
3. Same classification + same target issue/PR AND the previous directive was processed within the last 2 cycles

#### 6.3.2 Suppression Behavior

| Phase | Behavior |
|-------|---------|
| Report-Only | Log duplicate detection; process normally |
| Warning | Log and flag in cycle summary; process normally |
| Enforcement | Suppress duplicate; log suppression; do not re-process |

### 6.4 Pending Capability Gap Lifecycle

#### 6.4.1 Lifecycle States

```
REPORTED → ACKNOWLEDGED → PLANNED → IMPLEMENTED → VERIFIED → CLOSED
    │           │            │           │            │          │
    │           │            │           │            │          └─ Gap is resolved and verified
    │           │            │           │            └─ Implementation verified working
    │           │            │           └─ Implementation in progress
    │           │            └─ Plan exists for implementation
    │           └─ Controller has acknowledged the gap
    └─ Gap reported by agent (capability_gap_pending)
```

#### 6.4.2 Tracking Requirements

- Every capability gap must be assigned a tracking ID
- Gaps must have a severity (P0-P3) and estimated impact
- Gaps must be reviewed every 3 cycles (PM heartbeat)
- Gaps that remain in REPORTED state for > 5 cycles must be escalated to L2

### 6.5 PM Quiet-Cycle Behavior

#### 6.5.1 Quiet Cycle Rules

| Consecutive Quiet Cycles | Action |
|-------------------------|--------|
| 1-2 | Normal: Log quiet cycle reason; continue |
| 3 | Alert: Post summary to active anchor with reason analysis |
| 4 | Escalate: Post L2 planning request if no active directive |
| 5+ | Hold: PM enters standby; requires Controller wake |

#### 6.5.2 Quiet Cycle Reason Categories

| Reason | Description |
|--------|------------|
| NO_NEW_DIRECTIVE | No new ACTION REQUIRED comment found |
| DEDUPE_BLOCKED | All directives already processed (dedupe) |
| GATE_WAIT | Waiting for PR merge, Controller decision, or review |
| COORDINATOR_RELAY_CONSUMED | Relay comments processed; no new work |
| PLANNING_REQUEST_PENDING | L2/L3 request posted; awaiting response |

### 6.6 Dashboard Refresh Failure Reporting

Standardized failure report format:

```markdown
## Dashboard Refresh Failure Report

- **Timestamp**: <ISO-8601>
- **Cycle**: <N>
- **Status**: PARTIAL | FAILED
- **Failure Category**: <category>

### Details
- <specific failure description>

### Affected Repos
- <repo>: <error>

### Recovery Action
- <what was done to recover>

### Next Scheduled Refresh
- <ISO-8601>
```

### 6.7 Stream D Phase Plan

| Phase | Deliverable | Prerequisites | Activation Criteria |
|-------|------------|---------------|-------------------|
| **0: Planning** | This plan; operational rules documented | ✅ Complete | Controller approval |
| **1: Report-Only** | PM/Operator log cooldown violations, duplicate detections, quiet cycle reasons | Log format defined | Controller authorizes operational logging |
| **2: Warning** | PM/Operator post warnings for rule violations; capability gap tracking active | 2+ weeks of operational data; violation patterns cataloged | Controller authorizes warning mode |
| **3: Enforcement** | Hard blocks on cooldown violations; duplicate suppression; quiet-cycle standby | 2+ weeks of warning data; rules validated | Controller + Human authorize enforcement |

### 6.8 Risk Assessment — Stream D

| Risk | Severity | Mitigation |
|------|---------|-----------|
| Over-suppression of legitimate escalations | High | Conservative cooldown values; warning phase validates rules |
| Quiet-cycle standby causes missed directives | Medium | Controller wake mechanism; periodic re-check even in standby |
| Capability gap tracking adds overhead | Low | Lightweight tracking (JSON file); no external system |
| Operational rules conflict with existing AGENTS.md | Low | Rules supplement AGENTS.md; conflicts resolved during planning review |

---

## 7. Cross-Stream Dependencies

### 7.1 Dependency Matrix

| Stream | Depends On | Reason |
|--------|-----------|--------|
| A (Classification Registry) | B (Spectral CI) | Spectral validates registry file format (CLASS rules) |
| B (Spectral CI) | A (Classification Registry) | Registry provides classification naming rules for CLASS lint rules |
| C (State Consistency) | A (Classification Registry) | Dedupe keys reference classification names |
| D (Operational Hardening) | C (State Consistency) | Operational rules rely on state consistency (dedupe, dispatch ledger) |
| D (Operational Hardening) | A (Classification Registry) | Duplicate directive suppression uses classification matching |

### 7.2 Recommended Activation Order

```
Phase 0 (Planning):    A, B, C, D simultaneously (this document)
                          │
Phase 1 (Report-Only): A first → B second → C third → D fourth
                          │
Phase 2 (Warning):     B first (Spectral is lowest risk) → A second → C third → D fourth
                          │
Phase 3 (Enforcement): B first (Spectral required check) → A second → D third → C fourth
```

**Rationale**:
- Stream B (Spectral CI) has the lowest risk and highest immediate value — it catches document format issues without affecting agent behavior.
- Stream A (Classification Registry) should be in report-only before Spectral can validate CLASS rules effectively.
- Stream C (State Consistency) is the highest risk and should be activated last for enforcement.
- Stream D (Operational Hardening) depends on both A and C being stable.

### 7.3 Blocking Dependencies

| If Stream... | Is blocked at... | Then Stream... | Cannot advance to... |
|-------------|-----------------|---------------|---------------------|
| A | Phase 1 | B | Phase 2 (CLASS rules need registry data) |
| B | Phase 0 | A | Phase 3 (no CI validation of registry file) |
| C | Phase 1 | D | Phase 3 (operational rules need consistent state) |

---

## 8. Risk Assessment

### 8.1 Program-Level Risks

| Risk | Severity | Likelihood | Mitigation |
|------|---------|-----------|-----------|
| Hardening introduces more bugs than it prevents | High | Low | Graduated phases; extensive observation periods |
| Enforcement blocks critical urgent work | High | Medium | Emergency override mechanism (Controller can temporarily disable checks) |
| Too many warnings cause alert fatigue | Medium | High | Warning threshold tuning; warning deduplication |
| Hardening slows down dispatch cycles | Medium | Medium | Performance monitoring; async checks where possible |
| Cross-stream dependency creates deadlock | Low | Low | Independent phase advancement; blocking deps are soft |

### 8.2 Emergency Override

In all phases, the Controller retains emergency override authority:

- **Temporary suspension**: Controller can post `HARDENING_SUSPEND <stream> <duration>` to temporarily disable checks.
- **Emergency bypass**: Controller can post `HARDENING_BYPASS <stream> <scope>` to bypass checks for a specific operation.
- **Full rollback**: Controller can revert any stream to a previous phase.

All overrides are logged in the audit trail and require post-incident review.

---

## 9. Activation Criteria Per Stream

### 9.1 Summary Table

| Stream | Phase 0 → 1 | Phase 1 → 2 | Phase 2 → 3 |
|--------|-------------|-------------|-------------|
| **A: Classification Registry** | Registry YAML published; spec approved | 2+ weeks report-only; false-positive rate < 5% | 2+ weeks warning; all known mismatches resolved; Controller + Human approval |
| **B: Spectral CI** | Spectral config published; ruleset finalized | Baseline audit complete; violation count documented; ruleset stable 2+ weeks | All ERROR violations resolved or grandfathered; 2+ weeks warning soak; Controller + Human approval |
| **C: State Consistency** | Audit log format defined; logging infrastructure ready | 2+ weeks audit data; common inconsistency patterns cataloged | WAL implementation complete; recovery tested; 2+ weeks warning soak; Controller + Human approval |
| **D: Operational Hardening** | Log format defined; operational rules documented | 2+ weeks operational data; violation patterns cataloged; rules validated | 2+ weeks warning data; cooldown values validated; Controller + Human approval |

### 9.2 Cross-Cutting Activation Requirements

Before any stream moves to Phase 3 (Enforcement):

1. **Soak period**: Minimum 2 weeks in Phase 2 (Warning) with acceptable metrics.
2. **Rollback plan**: Documented procedure to revert to Phase 2 within 5 minutes.
3. **Emergency override**: Controller override mechanism tested.
4. **Audit trail**: All enforcement actions produce audit entries.
5. **Dashboard visibility**: Dashboard shows enforcement status for all streams.

---

## 10. Compliance and Audit Trail

### 10.1 Compliance with Existing Governance

| Existing Spec | How This Plan Complies |
|--------------|----------------------|
| **Enforcement Activation Policy** | This plan follows the same progressive activation model; each stream is a scoped activation with Controller gating |
| **Evolution Guardrail Spec** | This plan is a planning document (§2.2: new governance spec requires Controller authorization); no frozen artifacts are modified |
| **Execution Enforcement Phase Model** | This plan does not execute any work; it plans future enforcement that will operate within the phase model |
| **Classification Registry Spec Plan** | Stream A implements the registry runtime behavior described in the spec plan |
| **Verification Hardening Spec** | Stream B (Spectral CI) aligns with verification pipeline requirements |
| **Failure Replay Model** | Stream C (State Consistency) incorporates crash recovery and replay protection from the failure replay model |

### 10.2 Audit Trail Requirements

Every hardening phase transition must produce:

| Artifact | Description |
|---------|------------|
| Authorization comment | Controller comment authorizing the phase transition |
| Baseline report | Report showing metrics at the time of transition |
| Rollback plan | Documented procedure to revert |
| Soak period log | Log of metrics during the observation period |
| Transition record | Entry in `IEF-Orchestration/logs/hardening_transitions.jsonl` |

### 10.3 Hardening Transition Log Format

```jsonl
{
  "ts": "2026-06-28T12:00:00Z",
  "stream": "A|B|C|D",
  "from_phase": 0,
  "to_phase": 1,
  "authorization_comment_id": 0,
  "authorization_actor": "controller",
  "baseline_metrics": {},
  "rollback_plan_ref": "docs/governance/SYSTEM_HARDENING_PLAN.md#rollback",
  "notes": ""
}
```

---

## Appendix A: No Runtime Behavior Changed Statement

**This PR creates exactly one file: `docs/governance/SYSTEM_HARDENING_PLAN.md`.**

The following are explicitly NOT included and NOT changed:

- ❌ No registry runtime enforcement code
- ❌ No PM classification handler modifications
- ❌ No Spectral CI configuration files
- ❌ No CI workflow files
- ❌ No WAL implementation code
- ❌ No state file format changes
- ❌ No agent prompt or AGENTS.md modifications
- ❌ No dashboard.html modifications
- ❌ No dispatch logic changes
- ❌ No build-breaking checks

All hardening streams require separate PRs with separate Controller authorization for each phase transition.

---

## Appendix B: Follow-Up Recommendations

After this plan is approved, the recommended next actions are:

1. **Stream A Phase 1**: Create `CLASSIFICATION_REGISTRY.yaml` (v0.1.0) with the 13 known active classifications. Separate PR, separate authorization.
2. **Stream B Phase 1**: Create `.spectral.yaml` with the ruleset baseline. Separate PR, separate authorization.
3. **Stream C Phase 1**: Define audit log format and begin logging state mutations. Separate PR, separate authorization.
4. **Stream D Phase 1**: Define operational log format and begin logging cooldown/dedupe/quiet-cycle events. Separate PR, separate authorization.

Each follow-up PR should reference this plan and the specific stream/phase it activates.

---

## Appendix C: Glossary

| Term | Definition |
|------|-----------|
| **Hardening stream** | An independent work track within the hardening plan (A, B, C, or D) |
| **Phase** | A stage in the progressive enforcement model (Planning, Report-Only, Warning, Enforcement) |
| **Soak period** | A minimum observation time in a phase before advancing to the next |
| **WAL** | Write-Ahead Log — a transaction log written before the actual state change |
| **Dedupe key** | A unique identifier derived from event context to prevent duplicate processing |
| **Preflight check** | A validation performed before dispatching work |
| **Cooldown** | A minimum wait time between escalation actions |
| **Grandfather** | To exempt existing violations from new rules while enforcing them on new artifacts |
| **Emergency override** | Controller mechanism to temporarily disable enforcement |
| **Baseline audit** | A comprehensive check of existing artifacts against new rules to establish a starting metric |

---

*This document is a planning artifact. It defines the path to system hardening but does not implement any of the hardening measures described. Each stream and phase transition requires separate Controller authorization and separate PRs.*
