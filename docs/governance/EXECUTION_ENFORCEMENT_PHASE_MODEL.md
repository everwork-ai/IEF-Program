# Execution Enforcement Phase Model v0

> **Status:** Design Only — No CI Implementation  
> **Created:** 2026-06-26T16:00+08:00  
> **Authority:** Program Controller POST_MERGE_NEXT_PHASE_DIRECTIVES (P1)  
> **Baseline:** IEF-Program `main` @ `57df718`  
> **Governance Profile:** Contract-Critical  
> **Companion docs:** `CLASSIFICATION_REGISTRY_SPEC_PLAN.md`, `IEF_PROGRAM_CONTROLLER_MODE.md`, `IEF_OPERATING_MODEL.md`, `IEF_AGENT_RULES.md`, `IEF_DEFINITION_OF_DONE.md`

---

## 1. Purpose

This document defines the **five-phase execution gating lifecycle** for the IEF program. Every unit of work — task, dispatch, contract creation, governance action — must pass through these phases in strict sequential order. No phase may be skipped. No phase may be merged with another.

The phase model enforces the principle that **all execution is gated, all gating is evidence-based, and all evidence is GitHub-durable**.

```
Report → Validate → Authorize → Execute → Verify/Close
  (0)      (1)        (2)        (3)        (4)
```

This is a **governance design document only**. It does not implement CI pipelines, runtime enforcement, agent logic, or automated blocking. Those are deferred to future implementation phases.

---

## 2. Scope

### 2.1 What This Model Governs

The phase model applies to all **execution-bearing work** within the IEF program:

| Work Category | Examples |
|---|---|
| Dispatch cycles | Operator worker cycles triggered by PM dispatch |
| Contract creation | New OpenAPI specs, JSON schemas, governance documents |
| Governance actions | Classification registry changes, phase model updates |
| Cross-repo operations | Multi-repo coordination, propagation directives |
| Smoke tests and diagnostics | L3 closed-loop tests, infrastructure diagnosis |

### 2.2 What This Model Does NOT Govern

- Pure read operations (status checks, heartbeat scans, classification consumption)
- Documentation edits that do not change governance behavior (typo fixes, formatting)
- Agent-internal state management (memory, session context)
- Human Owner direct actions (override authority, see Section 11)

### 2.3 Relationship to IEF Core Principles

From `IEF_OPERATING_MODEL.md`:

> No issue, no official task.  
> No linked PR, no official delivery.  
> No IEF Command Center entry, no program visibility.  
> No RFC/ADR, no major architecture decision.

The phase model extends these principles with execution gating:

> **No execution without authorization.**  
> **No authorization without validation.**  
> **No validation without reporting.**  
> **No close without verification.**

---

## 3. Phase Definitions

### Phase 0 — Report

**Characterization:** Work is identified and reported. No action is taken.

| Attribute | Value |
|---|---|
| **Phase state** | Work item exists as a report or proposal |
| **Authority required** | Any agent or human (L0–L4) |
| **Actions allowed** | Create issue, post comment, classify work item, link to IEF Command Center |
| **Actions forbidden** | Create branch, write code, modify files, dispatch work |
| **Evidence produced** | Issue creation, classification label, initial context comment |
| **Classification signals** | `IEF_BLOCKED`, `IEF_STAGE_READY`, `PROG_*` classification proposals |

**Entry criteria:**
- A work need is identified (by human, agent, or automated detection)
- A GitHub issue exists or is created
- The issue is linked to IEF Command Center

**Exit criteria (to Phase 1):**
- Work item is fully described with:
  - Target repo
  - Scope definition
  - Acceptance criteria
  - Known dependencies and blockers
- Initial classification assigned
- No outstanding `IEF_BLOCKED` on this specific work item

**Artifacts produced:**
- GitHub issue with complete description
- Classification label(s)
- Context comment with scope and dependencies

**Anti-skip constraint:**
- Phase 0 cannot be skipped. Every work item MUST have a GitHub issue before proceeding.
- An agent that attempts to create a branch or modify files without a Phase 0 report is in violation.

---

### Phase 1 — Validate

**Characterization:** Work item is validated for feasibility, scope, classification correctness, and dependency resolution.

| Attribute | Value |
|---|---|
| **Phase state** | Work item is under validation |
| **Authority required** | Coordinator (L2) or Controller (L1) for validation sign-off |
| **Actions allowed** | Inspect repos, check dependencies, validate classifications, post validation reports |
| **Actions forbidden** | Create working branch, modify target files, dispatch execution |
| **Evidence produced** | Validation report, dependency check results, classification verification |
| **Classification signals** | `OPS_ACTION_REQUIRED` (validated, ready for authorization), `OPS_WAITING_*` (blocked on dependency) |

**Entry criteria (from Phase 0):**
- Exit criteria from Phase 0 met
- Work item has complete issue description
- Classification assigned and validated against registry

**Exit criteria (to Phase 2):**
- Validation report posted to issue with:
  - Scope confirmed feasible
  - Dependencies resolved or explicitly acknowledged
  - Classification confirmed correct per registry
  - Target repo and branch identified
  - Allowed files enumerated
  - Forbidden actions enumerated
- No unresolved `IEF_BLOCKED` or `OPS_WAITING_*` on this work item
- Validator (Coordinator or Controller) signs off

**Artifacts produced:**
- Validation report comment on issue
- Trigger JSON (if dispatch-based work)
- Branch name and base branch confirmed
- Allowed files list
- Forbidden actions list

**Anti-skip constraint:**
- Phase 1 cannot be skipped. A work item MUST be validated before authorization.
- An agent that attempts to seek authorization without a Phase 1 validation report is in violation.
- Validation must be performed by a different actor than the one who created the Phase 0 report (separation of concerns).

---

### Phase 2 — Authorize

**Characterization:** Validated work is authorized for execution by the appropriate authority level.

| Attribute | Value |
|---|---|
| **Phase state** | Work item is authorized, awaiting execution |
| **Authority required** | Controller (L1) for execution authorization; Human Owner (L0) for high-risk items |
| **Actions allowed** | Post authorization comment, create trigger JSON, set priority, define timeout |
| **Actions forbidden** | Begin execution, create branch, modify target files |
| **Evidence produced** | Authorization comment with `Label: ACTION REQUIRED`, trigger JSON |
| **Classification signals** | `PROG_AUTHORIZATION_GRANTED`, `OPS_ACTION_REQUIRED` (with authorization reference) |

**Entry criteria (from Phase 1):**
- Exit criteria from Phase 1 met
- Validation report exists and is signed off
- Trigger JSON created (if dispatch-based)
- All dependencies resolved

**Exit criteria (to Phase 3):**
- Authorization comment posted to issue/PR with:
  - `Label: ACTION REQUIRED`
  - Explicit scope (target repo, branch, allowed files)
  - Explicit constraints (forbidden actions, hard constraints)
  - Authority chain reference (Controller decision, relay comment)
  - Priority level (P0, P1, P2)
  - Timeout value
- Trigger JSON exists and is valid (if dispatch-based)
- Authorizing actor has sufficient authority for the work category

**Artifacts produced:**
- Authorization comment (GitHub issue/PR comment with `Label: ACTION REQUIRED`)
- Trigger JSON file (if dispatch-based)
- Priority and timeout metadata

**Authorization matrix:**

| Work Category | Minimum Authority | Additional Requirements |
|---|---|---|
| Contract creation (governance specs) | L1 (Controller) | Must reference Controller decision |
| Dispatch execution (worker cycles) | L1 (Controller) delegated to L2 (PM) | Must reference trigger JSON |
| Schema modification | L1 (Controller) | Must reference Controller decision |
| Classification vocabulary change | L0 (Human Owner) or L1 (Controller) | Must update registry |
| Cross-repo coordination | L1 (Controller) | Must list all target repos |
| Smoke test / diagnostic | L2 (PM/Coordinator) | Must reference test scope |

**Anti-skip constraint:**
- Phase 2 cannot be skipped. No execution may begin without explicit authorization.
- Self-authorization is forbidden: the authorizing actor must be different from the executing actor.
- An agent that begins execution without a Phase 2 authorization comment is in violation.
- Authorization is **scope-bound**: the authorization covers only the files, branches, and actions specified. Any deviation requires re-authorization.
- Authorization is **time-bound**: if the timeout expires before execution begins, the authorization lapses and Phase 2 must be re-entered.

---

### Phase 3 — Execute

**Characterization:** Authorized work is executed within the defined scope and constraints.

| Attribute | Value |
|---|---|
| **Phase state** | Work is being executed |
| **Authority required** | Operator (L3) or Runner (L3) as specified in authorization |
| **Actions allowed** | Create branch, modify allowed files only, commit, push, post progress comments |
| **Actions forbidden** | Modify non-allowed files, merge PRs, close issues, expand scope, perform forbidden actions |
| **Evidence produced** | Commits, branch push, worker output, progress comments |
| **Classification signals** | `RUN_EXECUTABLE_IDLE` (execution started), `RUN_ARTIFACT_DELIVERED` (execution completed) |

**Entry criteria (from Phase 2):**
- Exit criteria from Phase 2 met
- Authorization comment exists with `Label: ACTION REQUIRED`
- Trigger JSON valid (if dispatch-based)
- Executor validated the authorization (re-read directive, confirm scope)
- Working tree clean on target branch

**Exit criteria (to Phase 4):**
- Execution completed within scope:
  - Only allowed files modified
  - No forbidden actions performed
  - Commits pushed to target branch
  - Worker exit code 0 (or documented failure)
- Delivery report drafted (content prepared, not yet posted)
- Changed files verified against allowed files list
- Executor confirms no scope expansion occurred

**Artifacts produced:**
- Git branch with commits
- Pushed to remote (origin)
- Delivery report (prepared, not posted)
- Execution log (worker output, exit code, duration)

**Execution rules:**
1. **One cycle per authorization.** The operator may execute exactly one bounded worker cycle per Phase 2 authorization. No retries without new authorization.
2. **Scope adherence.** The executor must verify changed files are within the allowed files list before committing.
3. **Forbidden action enforcement.** The executor's allowed-tools whitelist must match the forbidden actions list. Any denial is logged.
4. **Timeout enforcement.** If the timeout expires during execution, the executor must stop and report failure.
5. **No self-unblock.** If execution is blocked (dependency failure, validation error, tool failure), the executor must report and stop. It may not unblock itself.

**Anti-skip constraint:**
- Phase 3 cannot be skipped. Authorized work must be executed (or explicitly declined with reason).
- An agent that posts a delivery report without Phase 3 execution evidence (commits, branch) is in violation.
- Execution must produce tangible artifacts (commits). A "no-op execution" (no changes needed) must still document why no changes were required.

---

### Phase 4 — Verify/Close

**Characterization:** Executed work is verified against done criteria, delivery report is posted, and the work item is closed or returned for revision.

| Attribute | Value |
|---|---|
| **Phase state** | Work is complete, awaiting verification and closure |
| **Authority required** | Coordinator (L2) for verification; Controller (L1) or Human Owner (L0) for close on high-risk items |
| **Actions allowed** | Post delivery report, update dispatch ledger, close issue (if authorized), request review |
| **Actions forbidden** | Modify delivered files (unless revision requested), create new branches, expand scope |
| **Evidence produced** | Delivery report comment, ledger update, verification checklist |
| **Classification signals** | `RUN_ARTIFACT_DELIVERED`, `OPS_READY_FOR_PROGRAM_REVIEW`, `KNW_REVIEW_RETURNED` |

**Entry criteria (from Phase 3):**
- Exit criteria from Phase 3 met
- Execution artifacts exist (commits, branch pushed)
- Delivery report content prepared
- Changed files verified against allowed files

**Exit criteria (work item complete):**
- Delivery report posted to target issue/PR with:
  - Trigger ID and directive comment reference
  - Previous and new head SHA
  - Files changed (verified against allowed files)
  - Worker exit status
  - Forbidden actions verified NOT taken
  - Checks run and results
- Dispatch ledger updated with:
  - Status (SUCCESS, FAILED, BLOCKED)
  - New head SHA
  - Delivery report URL
  - Evidence chain
- Work item classified as:
  - `SUCCESS` — all done criteria met, work complete
  - `FAILED` — execution failed, requires re-authorization for retry
  - `BLOCKED` — external dependency prevents completion
  - `NEEDS_REVISION` — delivered but does not meet done criteria

**Verification checklist:**

| Check | Required | Verifier |
|---|---|---|
| Delivery report posted | Yes | Executor (posts), Coordinator (verifies) |
| Files changed ⊆ allowed files | Yes | Coordinator |
| Forbidden actions not taken | Yes | Coordinator |
| Commit pushed to correct branch | Yes | Coordinator |
| Head SHA matches expected | Yes | Coordinator |
| Dispatch ledger updated | Yes | Executor |
| Done criteria evaluated | Yes | Coordinator or Controller |
| PR opened (if applicable) | Yes | Executor or Coordinator |

**Closure rules:**

| Result | Closure Action | Authority |
|---|---|---|
| SUCCESS | Issue may be closed (if no PR required) or PR opened for review | Coordinator (L2) or Controller (L1) |
| FAILED | Issue remains open; re-authorization required for retry | Controller (L1) must issue new Phase 2 |
| BLOCKED | Issue remains open; dependency resolution required | Coordinator (L2) tracks dependency |
| NEEDS_REVISION | Issue remains open; revision directive issued | Controller (L1) issues new Phase 2 with revision scope |

**Anti-skip constraint:**
- Phase 4 cannot be skipped. Every execution MUST be followed by verification and closure.
- An agent that marks work as complete without posting a delivery report is in violation.
- Closure authority must be different from execution authority (separation of concerns).
- A `SUCCESS` result on a Contract-Critical governance document requires Controller (L1) review before the issue may be closed.

---

## 4. Phase Transition Rules

### 4.1 Forward Transitions

| From | To | Trigger | Authority | Evidence |
|---|---|---|---|---|
| Phase 0 → Phase 1 | Report → Validate | Issue complete, classified | Coordinator (L2) | Issue description, classification |
| Phase 1 → Phase 2 | Validate → Authorize | Validation signed off | Coordinator (L2) or Controller (L1) | Validation report |
| Phase 2 → Phase 3 | Authorize → Execute | Authorization comment posted | Controller (L1) or delegate | Authorization comment, trigger JSON |
| Phase 3 → Phase 4 | Execute → Verify/Close | Execution completed | Executor (L3) | Commits, branch push, delivery report draft |

### 4.2 Backward Transitions (Return for Rework)

| From | To | Trigger | Authority | Recovery |
|---|---|---|---|---|
| Phase 1 → Phase 0 | Validate → Report | Validation failed (scope unclear, dependencies unresolved) | Coordinator (L2) | Revise issue description, re-classify |
| Phase 2 → Phase 1 | Authorize → Validate | Authorization declined or scope changed | Controller (L1) | Re-validate with updated scope |
| Phase 3 → Phase 2 | Execute → Authorize | Execution failed, re-authorization needed | Controller (L1) | New authorization with updated constraints |
| Phase 4 → Phase 3 | Verify/Close → Execute | Verification found issues (NEEDS_REVISION) | Controller (L1) | Revision directive, re-execute within scope |
| Phase 4 → Phase 0 | Verify/Close → Report | Fundamental problem discovered | Controller (L1) or Human Owner (L0) | Full re-scoping required |

### 4.3 Lapse Conditions

A phase authorization lapses when:

| Condition | Effect |
|---|---|
| Timeout expires before execution begins | Phase 2 authorization lapses; must re-authorize |
| Target branch has new commits not in authorization | Phase 2 authorization lapses; must re-validate and re-authorize |
| Directive comment is edited or deleted | Phase 2 authorization lapses; must re-issue |
| Classification changes after authorization | Phase 2 authorization lapses; must re-validate |

---

## 5. Anti-Skip Constraints

### 5.1 Universal Anti-Skip Rules

These rules are **absolute and non-negotiable**:

| Rule | Description | Violation Severity |
|---|---|---|
| **AS-1** | No phase may be skipped. Work must pass through all five phases sequentially. | Critical |
| **AS-2** | No phase may be merged with another. Each phase has distinct entry/exit criteria. | Critical |
| **AS-3** | No agent may self-promote its work to the next phase. Phase transitions require independent validation. | Critical |
| **AS-4** | No agent may self-authorize. The authorizing actor must be different from the executing actor. | Critical |
| **AS-5** | No agent may self-verify. The verifying actor must be different from the executing actor. | Critical |
| **AS-6** | No execution without a GitHub-durable authorization comment. Chat-only authorization is invalid. | Critical |
| **AS-7** | No closure without a GitHub-durable delivery report. Chat-only completion is invalid. | Critical |
| **AS-8** | No retry without new authorization. A failed execution requires a new Phase 2 cycle. | High |
| **AS-9** | No scope expansion during execution. If scope needs to change, return to Phase 1 or Phase 2. | High |
| **AS-10** | No phase transition without evidence. Every transition must produce the artifacts specified in the phase definition. | High |

### 5.2 Skip Detection

Phase skips are detected by checking for the presence of required artifacts:

| Expected Artifact | Missing Means |
|---|---|
| GitHub issue with classification | Phase 0 was skipped |
| Validation report comment | Phase 1 was skipped |
| Authorization comment with `Label: ACTION REQUIRED` | Phase 2 was skipped |
| Git commits on target branch | Phase 3 was skipped or was no-op without documentation |
| Delivery report comment | Phase 4 was skipped |

### 5.3 Skip Response

When a phase skip is detected:

1. **Immediate halt**: The work item is flagged as `PHASE_SKIP_DETECTED`.
2. **Notification**: Post a violation report to the issue identifying the skipped phase.
3. **Rollback**: Return the work item to the last valid phase.
4. **Ledger entry**: Record the violation in the dispatch ledger.
5. **Controller notification**: Controller must acknowledge before work resumes.

### 5.4 Known Anti-Patterns (Historical)

These patterns have been observed and must be prevented by the phase model:

| Anti-Pattern | Description | Phase Violated | Mitigation |
|---|---|---|---|
| **Silent execution** | Agent creates branch and commits without issue or authorization | Phase 0, 1, 2 | AS-1, AS-6 |
| **Self-dispatch** | Agent creates its own trigger JSON and executes it | Phase 2 | AS-4 |
| **Scope creep** | Agent modifies files outside allowed list during execution | Phase 3 | AS-9, execution rules |
| **Phantom completion** | Agent reports success without delivery report or commits | Phase 4 | AS-7 |
| **Retry spiral** | Agent retries failed execution without new authorization | Phase 2 | AS-8 |
| **Chat-only state** | Agent treats chat memory as durable state for phase transitions | All phases | AS-6, AS-7 |
| **Classification bypass** | Agent uses undefined classification to skip validation | Phase 1 | Classification registry preflight |
| **Dedupe key survival** | Stale dedupe key allows re-processing of already-completed work | Phase 2/3 | Dedupe key invalidation on state change (per Classification Registry) |

---

## 6. Authority and Delegation

### 6.1 Authority Levels

| Level | Actor | Phase Authority |
|---|---|---|
| **L0** | Human Owner | All phases; may override any constraint with explicit decision |
| **L1** | Program Controller | Phase 0–4; authorizes Phase 2 for execution; reviews Phase 4 for Contract-Critical |
| **L2** | Coordinator / PM | Phase 0–2; validates Phase 1; creates trigger JSON; verifies Phase 4 |
| **L3** | Operator / Runner | Phase 3 execution only; may not authorize or verify own work |
| **L4** | External Adapter | Phase 0 reporting only; may not validate, authorize, or execute |

### 6.2 Delegation Rules

- L1 (Controller) may delegate Phase 2 authorization to L2 (Coordinator/PM) for specific bounded tasks.
- Delegation is **task-scoped**: the delegate may authorize only the specific work item defined in the delegation directive.
- Delegation is **non-transferable**: the delegate may not further delegate.
- Delegation **expires** after one execution cycle or the trigger timeout, whichever comes first.
- Delegation must be **GitHub-durable**: recorded in a directive comment with explicit scope.

### 6.3 Separation of Concerns

| Role | May NOT also be |
|---|---|
| Phase 0 reporter | Phase 1 validator of the same work item |
| Phase 1 validator | Phase 2 authorizer of the same work item |
| Phase 2 authorizer | Phase 3 executor of the same work item |
| Phase 3 executor | Phase 4 verifier of the same work item |

**Exception:** For low-risk work (documentation edits, typo fixes), the Controller may waive separation of concerns with explicit rationale.

---

## 7. Classification Integration

The phase model uses classifications from the Classification Registry (`CLASSIFICATION_REGISTRY_SPEC_PLAN.md`) to signal phase state.

### 7.1 Phase-to-Classification Mapping

| Phase | Primary Classification | Secondary Classifications |
|---|---|---|
| Phase 0 (Report) | `IEF_STAGE_READY`, `IEF_BLOCKED` | `PROG_*` proposals |
| Phase 1 (Validate) | `OPS_WAITING_*`, `OPS_NO_SAFE_ACTION` | `OPS_WAITING_CODEX`, `OPS_WAITING_AGENT`, `OPS_WAITING_HUMAN` |
| Phase 2 (Authorize) | `PROG_AUTHORIZATION_GRANTED`, `OPS_ACTION_REQUIRED` | Priority metadata (P0, P1, P2) |
| Phase 3 (Execute) | `RUN_EXECUTABLE_IDLE`, `RUN_ARTIFACT_DELIVERED` | `OPS_NO_SAFE_ACTION` (if blocked during execution) |
| Phase 4 (Verify/Close) | `OPS_READY_FOR_PROGRAM_REVIEW`, `KNW_REVIEW_RETURNED` | `OPS_NO_ACTION_REQUIRED` (on SUCCESS) |

### 7.2 Classification Behavior by Phase

| Classification | Behavior Type | Phase |
|---|---|---|
| `PROG_AUTHORIZATION_GRANTED` | EXECUTABLE | Phase 2 (triggers Phase 3 readiness) |
| `OPS_ACTION_REQUIRED` | EXECUTABLE | Phase 2 (dispatches to executor) |
| `RUN_EXECUTABLE_IDLE` | EXECUTABLE | Phase 3 (execution started) |
| `RUN_ARTIFACT_DELIVERED` | ACK_ONLY | Phase 4 (execution completed) |
| `IEF_BLOCKED` | ACK_ONLY | Any phase (halts progression) |
| `IEF_UNBLOCKED` | EXECUTABLE | Any phase (resumes progression) |
| `OPS_NO_ACTION_REQUIRED` | ACK_ONLY | Phase 4 (work complete) |
| `OPS_NO_SAFE_ACTION` | ACK_ONLY | Phase 1 or 3 (safety check prevented action) |

### 7.3 Dedupe Integration

Per the Classification Registry dedupe rules:

- `PROG_AUTHORIZATION_GRANTED` uses `ERROR` strategy: duplicate authorization is rejected. This prevents double-execution from a single authorization.
- `OPS_ACTION_REQUIRED` uses `UPDATE` strategy: new dispatch context replaces stale. This allows re-dispatch if the first dispatch was not consumed.
- `IEF_BLOCKED` uses `IGNORE` strategy: already blocked, no re-block.
- Dedupe keys MUST be invalidated on phase transitions. A stale dedupe key from Phase 2 must not prevent Phase 3 execution of a genuinely new dispatch.

---

## 8. Evidence Chain Requirements

Every phase transition must produce GitHub-durable evidence. Chat-only evidence is insufficient.

### 8.1 Evidence Types

| Evidence Type | Format | Durability |
|---|---|---|
| GitHub issue | Issue creation | Permanent (GitHub-hosted) |
| GitHub comment | Issue/PR comment | Permanent (GitHub-hosted) |
| Classification label | Issue label or comment prefix | Permanent (GitHub-hosted) |
| Trigger JSON | File in `triggers/` directory | Durable (repo-hosted, may be ephemeral) |
| Git commit | Branch commit with SHA | Permanent (GitHub-hosted) |
| Delivery report | PR/issue comment | Permanent (GitHub-hosted) |
| Dispatch ledger | `state/last_dispatch.json` entry | Durable (repo-hosted) |

### 8.2 Evidence Chain by Phase

```
Phase 0: Issue → Classification → Context Comment
              ↓
Phase 1: Validation Report → Dependency Check → Sign-off Comment
              ↓
Phase 2: Authorization Comment → Trigger JSON → Priority/Timeout
              ↓
Phase 3: Branch → Commits → Push → Worker Output
              ↓
Phase 4: Delivery Report → Ledger Update → Verification Checklist
```

### 8.3 Broken Evidence Chain

If any link in the evidence chain is missing:

1. The phase transition is **invalid**.
2. The work item is flagged as `EVIDENCE_CHAIN_BROKEN`.
3. Work returns to the phase where evidence is missing.
4. Controller must acknowledge before work resumes.

---

## 9. Timeout and Expiration

### 9.1 Phase Timeouts

| Phase | Default Timeout | Timeout Action |
|---|---|---|
| Phase 0 (Report) | No timeout | Work item remains until validated or explicitly abandoned |
| Phase 1 (Validate) | 48 hours | Escalate to Controller if validation stalls |
| Phase 2 (Authorize) | Per trigger `timeout` field | Authorization lapses; re-validate and re-authorize |
| Phase 3 (Execute) | Per trigger `timeout` field (default 1500s) | Execution halted; report failure |
| Phase 4 (Verify/Close) | 24 hours | Escalate to Coordinator for verification |

### 9.2 Authorization Expiration

- Phase 2 authorizations have an explicit timeout in the trigger JSON.
- If execution does not begin before timeout, the authorization lapses.
- A lapsed authorization requires a new Phase 2 cycle (re-validation may be skipped if scope unchanged).

### 9.3 Stale Work Detection

A work item is considered stale if:

- It has been in the same phase for longer than the phase timeout.
- No comments or activity have occurred in 72 hours.
- The target branch has diverged significantly from the authorization baseline.

Stale work items are flagged during Coordinator heartbeat cycles and escalated to Controller.

---

## 10. Exception Handling

### 10.1 Phase Violation Response

| Violation | Detection | Response |
|---|---|---|
| Phase skip | Missing evidence artifact | Halt work, post violation report, return to last valid phase |
| Self-authorization | Same actor validates and authorizes | Halt work, require independent authorization |
| Self-verification | Same actor executes and verifies | Halt verification, require independent verifier |
| Scope expansion | Changed files ⊄ allowed files | Reject commit, return to Phase 2 for re-authorization |
| Forbidden action | Allowed-tools denial or post-hoc detection | Halt execution, post violation report |
| Stale authorization | Timeout expired | Lapse authorization, require new Phase 2 |
| Broken evidence chain | Missing required artifact | Flag as EVIDENCE_CHAIN_BROKEN, return to deficient phase |

### 10.2 Emergency Override

The Human Owner (L0) may override any phase constraint with an explicit GitHub-durable decision comment. This is the only mechanism for bypassing phase requirements.

Emergency override must include:
- Specific constraint being overridden
- Rationale
- Scope (which work item, which phase)
- Expiration (one-time or time-bounded)

### 10.3 Concurrent Work Items

Multiple work items may be in flight simultaneously, each at its own phase. However:

- An executor (L3) may only execute one Phase 3 work item at a time.
- A validator (L2) may validate multiple Phase 1 work items concurrently.
- The Controller (L1) may authorize multiple Phase 2 work items concurrently.

---

## 11. Human Owner Override

The Human Owner (L0) retains the ability to:

- Skip any phase for any work item with explicit rationale.
- Override any phase constraint.
- Close any work item regardless of phase state.
- Revoke any authorization.
- Reassign any work item to a different executor.

Human Owner overrides must be GitHub-durable (issue comment or PR review) and are logged in the dispatch ledger.

---

## 12. Compliance Checklist

For any work item, verify:

- [ ] Phase 0: GitHub issue exists with classification and scope
- [ ] Phase 1: Validation report exists with dependency check and sign-off
- [ ] Phase 2: Authorization comment exists with `Label: ACTION REQUIRED` and scope
- [ ] Phase 2: Trigger JSON valid (if dispatch-based)
- [ ] Phase 3: Execution within scope (allowed files only, no forbidden actions)
- [ ] Phase 3: Commits pushed to correct branch
- [ ] Phase 4: Delivery report posted with all required fields
- [ ] Phase 4: Dispatch ledger updated
- [ ] No phase was skipped (all evidence artifacts present)
- [ ] No self-authorization, self-validation, or self-verification occurred
- [ ] Authorization was not expired at time of execution
- [ ] Classification used is registered in the registry

---

## 13. Interaction with Companion Documents

| Document | Interaction |
|---|---|
| `CLASSIFICATION_REGISTRY_SPEC_PLAN.md` | Phase signals use registered classifications. Dedupe rules prevent double-execution. Preflight validation catches undefined classifications before Phase 1. |
| `IEF_PROGRAM_CONTROLLER_MODE.md` | Controller authority (L1) drives Phase 2 authorization. Controller mode defines when and how authorization is granted. |
| `IEF_OPERATING_MODEL.md` | Phase model extends the "no issue, no task" principle with "no authorization, no execution." |
| `IEF_AGENT_RULES.md` | Agent behavior rules align with phase constraints: start from issue (Phase 0), comment plan (Phase 1), deliver via PR (Phase 3-4). |
| `IEF_DEFINITION_OF_DONE.md` | Phase 4 verification evaluates done criteria defined in Phase 0 and validated in Phase 1. |
| IEF-Operations `EVOLUTION_GUARDRAIL_SPEC.md` | Phase model governs work flow; Guardrail governs spec mutation. Both require PR-gating and review-gating. |
| IEF-Operations `EXECUTION_ENFORCEMENT_PHASE_MODEL.md` | Operations phase model governs spec maturity (Draft→Active→Guarded→Enforced→Locked). This model governs work execution. Complementary, not overlapping. |

---

## 14. Future Work

1. **CI Integration**: Automated phase-skip detection via GitHub Actions (check for evidence artifacts before allowing PR merge).
2. **Phase Dashboard**: Visual tracking of work items by phase in IEF Command Center.
3. **Phase Metrics**: Time-in-phase, skip attempts, violation rates, timeout frequency.
4. **Automated Phase Transition**: When all exit criteria are met, automatically advance to next phase (with human approval gate for Phase 2→3).
5. **Phase Templates**: Issue templates that enforce phase-appropriate content at each stage.

---

*End of specification.*
