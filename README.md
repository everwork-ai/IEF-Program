# IEF-Program

IEF-Program is the **HQ / program control interface** for **IEF — Intelligent Employee Foundry**.

It is not a runtime capability layer. It is the project-of-projects control repository through which humans and AI agents understand, enter, coordinate, track, and close work across the six IEF capability repositories.

## What IEF-Program Owns

- Long-term program roadmap
- RFCs and architecture design records
- ADRs (Architecture Decision Records)
- GitHub operating model and workflow definitions
- AI agent working rules
- Cross-repository contract issues and coordination
- Program status reports and definition of done

## What IEF-Program Does Not Own

- Runtime governance rules (see `IEF-Governance`)
- Task state engine or run ledger (see `IEF-Operations`)
- Protocol schemas or object contracts (see `IEF-Protocol`)
- Concrete runner implementations (see `IEF-Runners`)
- Host adapter code (see `IEF-Adapters`)
- Knowledge storage or context packs (see `IEF-Knowledge`)

## Source of Truth

| Object | Purpose |
|---|---|
| **IEF Command Center** (GitHub Project) | Dynamic cross-repository project status |
| **IEF-Program** (this repo) | Long-term roadmap, RFCs, ADRs, operating model, startup plan |
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

## Control Documents

| Document | Purpose |
|---|---|
| [IEF Overview](docs/IEF_OVERVIEW.md) | System-family definition and layer summary |
| [Operating Model](docs/IEF_OPERATING_MODEL.md) | Source-of-truth model, role boundaries, workflow |
| [Repository Matrix](docs/IEF_REPO_MATRIX.md) | Repository-to-layer mapping with ownership |
| [Roadmap](docs/IEF_ROADMAP.md) | v0 milestones: Setup → Core Loop → Protocol → Hardening |
| [Startup Plan](docs/IEF_STARTUP_PLAN.md) | Bootstrap sequence and dependency order |
| [Agent Rules](docs/IEF_AGENT_RULES.md) | AI agent working rules across repos |
| [Program Controller Mode](docs/IEF_PROGRAM_CONTROLLER_MODE.md) | GitHub-first control plane, roles, L1/L2/L3 boundaries, SYNC_FROM_GITHUB |
| [GitHub Workflow](docs/IEF_GITHUB_WORKFLOW.md) | Issue → PR → review → merge flow |
| [Status Report](docs/IEF_STATUS_REPORT.md) | Current status of each repo |
| [Definition of Done](docs/IEF_DEFINITION_OF_DONE.md) | Acceptance criteria per governance profile |

## Core Rules

1. **No issue, no official task.**
2. **No linked PR, no official delivery.**
3. **No entry in IEF Command Center, no cross-repo visibility.**
4. All AI agents must report execution plan and completion evidence in the issue.
5. High-risk changes require review before merge.
6. IEF-Program is the HQ. IEF-Governance is not the project control center.
