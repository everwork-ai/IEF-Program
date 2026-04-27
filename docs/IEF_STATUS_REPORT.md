# IEF Status Report

This document tracks the current readiness of each IEF repository.

Last updated: 2026-04-27

## Repository Status

| Repository | Layer | Status | README | Docs | Code/Schemas | Tests | Notes |
|---|---|---|---|---|---|---|---|
| IEF-Program | HQ | Ready | Yes | Yes | N/A | N/A | Control interface initialized |
| IEF-Governance | Charter | Migrated | Yes | Yes | Partial | N/A | Needs bootstrap profiles |
| IEF-Knowledge | Library | Migrated | No | Partial | Partial | N/A | Needs context pack v0 |
| IEF-Operations | Dispatch | Skeleton | Yes | No | No | No | Empty except README |
| IEF-Protocol | Relay | Skeleton | Yes | No | No | No | Empty except README |
| IEF-Runners | Hands | Restructured | Yes | Yes | Yes | Yes | ClaudeCode runner migrated |
| IEF-Adapters | Gateway | Migrated | Yes | Partial | Partial | N/A | Needs adapter contract v0 |

## Legend

| Symbol | Meaning |
|---|---|
| Yes | Ready or complete |
| Partial | Needs work |
| No | Missing or not started |
| N/A | Not applicable to this layer |

## Next Actions

1. **IEF-Operations** — Define Task, Run, Workflow, state machine, run ledger (blocked by Protocol draft)
2. **IEF-Protocol** — Define AgentCardLite, TaskEnvelope, RunEvent, ArtifactRef, ContextRef (no blockers)
3. **IEF-Governance** — Define bootstrap governance profiles (no blockers)
4. **IEF-Knowledge** — Define context pack and run summary formats (blocked by Protocol + Operations draft)
5. **IEF-Adapters** — Define host adapter contract (blocked by Protocol + Operations draft)
6. **IEF-Runners** — Define runner interface v0 (blocked by Protocol + Operations draft)

## Blocker Map

```text
IEF-Program (ready)
├── IEF-Governance (ready to start)
├── IEF-Protocol (ready to start)
│   └── blocks IEF-Operations
│       └── blocks IEF-Runners
│       └── blocks IEF-Knowledge
│       └── blocks IEF-Adapters
```

## Dynamic Status

For real-time status, see **IEF Command Center** (GitHub Project).

This Markdown status report is a snapshot. Dynamic progress, owner agents, phases, and blockers are tracked in the GitHub Project, not here.
