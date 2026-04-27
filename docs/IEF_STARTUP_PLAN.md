# IEF Startup Plan

This document defines the bootstrap sequence and dependency order for bringing IEF from repository scaffolding to a working minimum operating loop.

## Bootstrap Philosophy

1. **Program first** — Establish IEF-Program as HQ before coordinating downstream work.
2. **Contracts before implementation** — Define Protocol and Operations drafts before Runners, Knowledge, and Adapters build against them.
3. **Governance from day one** — Use lightweight bootstrap profiles (Design-Lite, Contract-Critical, Implementation-Controlled).
4. **One closed loop before scaling** — Prove one task end-to-end before adding complexity.

## Phase 0 — Program Control Interface (Current)

**Goal:** IEF-Program is ready to coordinate work.

**Deliverables:**
- [x] Repository created
- [x] README defines HQ role
- [x] Operating model documented
- [x] Repository matrix defined
- [x] Roadmap with milestones
- [x] Agent working rules
- [x] Initial RFCs and ADRs
- [x] PR templates and issue templates
- [x] First program epic: Minimum Operating Loop
- [x] Cross-repo contract issues created

## Phase 1 — Governance and Protocol Contracts

**Goal:** Downstream repos know what rules and objects to use.

**Dependency order:**

```text
IEF-Program Epic
├── IEF-Governance Bootstrap Profiles
│   └── Design-Lite / Contract-Critical / Implementation-Controlled
├── IEF-Protocol Object Model
│   ├── AgentCardLite
│   ├── TaskEnvelope
│   ├── RunEvent
│   ├── ArtifactRef
│   └── ContextRef
```

**Governance profiles:**

| Profile | Use For | Requirements |
|---|---|---|
| Design-Lite | README/doc/stub updates | linked issue, acceptance criteria, PR, reviewer check |
| Contract-Critical | Protocol objects, lifecycle, interfaces, contracts | design doc, explicit I/O, explicit boundary, schema or example, cross-review, Program approval |
| Implementation-Controlled | Code, scripts, validators, runner logic | linked issue, design reference, PR, test or dry-run evidence, rollback plan, review before merge |

## Phase 2 — Operations and Runners

**Goal:** Tasks can be created, assigned, executed, and closed.

**Dependency order:**

```text
IEF-Operations Lifecycle
├── depends on Protocol draft
├── Task lifecycle v0
├── Run ledger v0
└── Approval checkpoint v0

IEF-Runners Interface
├── depends on Protocol + Operations draft
├── Runner interface v0
└── ClaudeCode runner boundary
```

## Phase 3 — Knowledge and Adapters

**Goal:** Runs produce reusable memory; host environments can enter IEF work.

**Dependency order:**

```text
IEF-Knowledge Context Pack
├── depends on Protocol + Operations draft
├── Context pack format v0
└── Run summary format v0

IEF-Adapters Contract
├── depends on Protocol + Operations draft
├── Host adapter contract v0
├── Codex adapter stub
└── OpenClaw adapter stub
```

## Phase 4 — Minimum Operating Loop

**Goal:** Prove one end-to-end task.

**Loop:**

```text
task created -> governed -> assigned -> executed -> artifact produced -> reviewed -> summarized -> closed
```

**Steps:**
1. Task created through IEF-Program
2. Governance profile selected from IEF-Governance
3. Task object follows IEF-Protocol
4. Task/run lifecycle follows IEF-Operations
5. Execution performed by IEF-Runners
6. Integration entry defined by IEF-Adapters
7. Summary/context captured by IEF-Knowledge
8. PR closes the issue

## Anti-Patterns

- Do not implement all capability repos independently.
- Do not invent local object formats before Protocol and Operations drafts exist.
- Do not treat IEF-Governance as the project control center.
- Do not maintain parallel status trackers outside IEF Command Center.
