# IEF-Program

IEF-Program is the project-of-projects control repository for **IEF — Intelligent Employee Foundry**.

## Source of Truth

| Object | Purpose |
|---|---|
| **IEF Command Center** (GitHub Project) | Dynamic cross-repository project status |
| **IEF-Program** (this repo) | Long-term roadmap, RFCs, ADRs, operating model |
| **Capability repositories** | Concrete implementation and delivery |
| **Issues** | Official work entry |
| **PRs** | Official delivery entry |

## Repository Family

| Repository | Codename | Responsibility |
|---|---|---|
| IEF-Program | HQ | Program control, RFCs, roadmap, decision log |
| IEF-Governance | Charter | Rules, gates, review, audit, promotion |
| IEF-Knowledge | Library | Knowledge, memory, experience compounding |
| IEF-Operations | Dispatch | Task, workflow, state, queue, run ledger |
| IEF-Protocol | Relay | Agent cards, messages, artifacts, handoffs |
| IEF-Runners | Hands | Claude Code, Qoder, Codex, OpenHands runners |
| IEF-Adapters | Gateway | Host runtime adapters |

## Core Rules

1. **No issue, no official task.**
2. **No linked PR, no official delivery.**
3. **No entry in IEF Command Center, no cross-repo visibility.**
4. All AI agents must report execution plan and completion evidence in the issue.
5. High-risk changes require review before merge.
