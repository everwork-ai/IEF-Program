# IEF GitHub Operating Model

## Source of Truth Model

| Object | Source of Truth For |
|---|---|
| `everwork-ai` organization | Ownership, access, repository namespace |
| `IEF Command Center` GitHub Project | Dynamic progress, status, owner agent, phase, blockers |
| `IEF-Program` repository | Long-term program design, roadmap, RFCs, ADRs, operating model |
| Capability repositories | Actual implementation, docs, code, tests, package assets |
| Issues | Work entry and work tracking |
| PRs | Delivery and review |
| Milestones | Phase/release grouping |

## Core Rule

```
No issue, no official task.
No linked PR, no official delivery.
No IEF Command Center entry, no program visibility.
No RFC/ADR, no major architecture decision.
```

## Dynamic Status

**IEF Command Center** is the dynamic status source of truth. Do not maintain parallel status trackers in chat or private documents.

## Long-Term Decisions

**IEF-Program** stores RFCs and ADRs. Decisions must be recorded before implementation begins.

## Agent Behavior

Every AI agent working on IEF must:

1. Start from a GitHub issue.
2. Ensure the issue is visible in IEF Command Center.
3. Comment with an execution plan before making changes.
4. Attach commits, PRs, artifacts, and review notes to the issue.
5. State blockers explicitly.
6. Avoid untracked cross-repository work.
7. Link every PR to an issue.

## Cross-Repository Contracts

```
Governance -> Operations: rules, evidence, approvals, promotion path
Knowledge -> Operations: similar work, playbooks, project memory, risks
Operations -> Runners: task slice, context pack, policy constraints, run ID
Runners -> Protocol: status events, artifacts, handoff payloads, errors
Adapters -> Operations: external requests -> internal IEF work objects
Governance -> Adapters: governance packs -> host-specific rules
Knowledge -> Governance / Adapters / Runners: context packs, patterns, decisions
```
