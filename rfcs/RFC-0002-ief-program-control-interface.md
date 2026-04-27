# RFC-0002: IEF-Program Control Interface

**Status:** Proposed

**Governance Profile:** Contract-Critical

## Problem

IEF has six capability repositories. Without a program control interface:
- Work enters repositories independently
- Cross-repo dependencies are invisible
- Agent behavior is inconsistent
- No single source of truth for program-level decisions
- Governance rules and project coordination are confused

## Proposal

Establish `IEF-Program` as the dedicated HQ / program control interface with explicit responsibilities, boundaries, and artifacts.

### Responsibilities

| Responsibility | Owner | Artifact |
|---|---|---|
| Program roadmap | IEF-Program | `docs/IEF_ROADMAP.md` |
| Operating model | IEF-Program | `docs/IEF_OPERATING_MODEL.md` |
| Agent working rules | IEF-Program | `docs/IEF_AGENT_RULES.md` |
| Architecture decisions | IEF-Program | `decisions/ADR-XXXX-*.md` |
| Design proposals | IEF-Program | `rfcs/RFC-XXXX-*.md` |
| Cross-repo coordination | IEF-Program | Issues in IEF-Program + IEF Command Center |
| Governance rules | IEF-Governance | `governance/` packs |
| Task state | IEF-Operations | Task/run objects |

### Boundaries

**IEF-Program does not own:**
- Runtime governance rule validation
- Task state machine logic
- Protocol schema enforcement
- Runner implementation code
- Adapter host bindings
- Knowledge storage semantics

**IEF-Program does own:**
- Which repo owns what
- How agents should work
- What "done" means per profile
- When a contract is ready for implementation

### Artifacts

Required control documents:
- `README.md` — HQ role declaration
- `docs/IEF_OVERVIEW.md` — system definition
- `docs/IEF_OPERATING_MODEL.md` — source-of-truth model
- `docs/IEF_REPO_MATRIX.md` — ownership matrix
- `docs/IEF_ROADMAP.md` — milestones
- `docs/IEF_STARTUP_PLAN.md` — bootstrap sequence
- `docs/IEF_AGENT_RULES.md` — agent behavior
- `docs/IEF_GITHUB_WORKFLOW.md` — workflow rules
- `docs/IEF_STATUS_REPORT.md` — current status
- `docs/IEF_DEFINITION_OF_DONE.md` — acceptance criteria

Required process artifacts:
- `epics/` — program-level epics
- `rfcs/` — design proposals
- `decisions/` — architecture decision records
- `.github/ISSUE_TEMPLATE/` — issue templates
- `.github/PULL_REQUEST_TEMPLATE.md` — PR template

## Alternatives Considered

| Alternative | Rejected Because |
|---|---|
| Use IEF-Governance as control center | Violates separation of governance rules and project coordination |
| Use IEF Command Center as sole control | GitHub Project is dynamic status only, not long-term decision storage |
| Spread control docs across all repos | Creates fragmentation; no single entry point |
| No control interface at all | Leads to untracked, uncoordinated work |

## Impact

- All AI agents must start from IEF-Program issues or linked capability repo issues
- All cross-repo work must be visible in IEF Command Center
- All major decisions require ADR in IEF-Program
- IEF-Governance is freed to focus on governance rules only

## Migration

This RFC replaces any implicit "governance as PMO" pattern.
- Existing ADRs remain valid
- IEF-Governance retains all governance packs
- IEF-Program takes ownership of coordination artifacts

## Open Questions

1. Should IEF-Program maintain a machine-readable repo matrix (JSON/YAML) in addition to Markdown?
2. Should agent rules be versioned per IEF release?
3. How often should the status report be updated?
