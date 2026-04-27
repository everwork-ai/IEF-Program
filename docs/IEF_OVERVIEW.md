# IEF Overview

**IEF — Intelligent Employee Foundry**

A system family for building governable, persistent, always-working enterprise AI employees.

## One-Sentence Definition

IEF is the control-and-work framework used to create AI workers that can receive tasks, operate under governance, collaborate through explicit protocols, produce auditable artifacts, and improve through knowledge feedback.

## The Six Capability Questions

IEF answers six recurring questions through six capability repositories:

1. **Governance** — What counts as acceptable work? (`IEF-Governance`)
2. **Knowledge** — What have we learned already? (`IEF-Knowledge`)
3. **Operations** — What work is being done now, by whom, and in what state? (`IEF-Operations`)
4. **Protocol** — How do AI workers communicate and hand off work? (`IEF-Protocol`)
5. **Runners** — Which concrete runners actually perform assigned work? (`IEF-Runners`)
6. **Adapters** — How do we inject IEF into different agent environments? (`IEF-Adapters`)

## The Seventh Question: Program

**Program** — How is the IEF project family planned, tracked, decided, and coordinated across repositories and AI agents? (`IEF-Program`)

Program is not a runtime capability. It is the coordination and control interface.

## Core Loop

```text
Request
  -> Operations task
  -> Governance lookup
  -> Knowledge context pack
  -> Runner selection
  -> Execution run
  -> Protocol status/artifact events
  -> Review / approval
  -> Completion / escalation
  -> Knowledge extraction
  -> Better future execution
```

## Enterprise-Worker Loop

```text
Assign work
  -> run under policy
  -> produce evidence
  -> review and decide
  -> retain memory
  -> improve the system
```

This loop is the minimum viable definition of an enterprise AI worker.

## Source of Truth Model

| Object | Source of Truth For |
|---|---|
| `everwork-ai` organization | Ownership, access, repository namespace |
| `IEF Command Center` GitHub Project | Dynamic progress, status, owner agent, phase, blockers |
| `IEF-Program` repository | Long-term program design, roadmap, RFCs, ADRs, operating model |
| Capability repositories | Actual implementation, docs, code, tests, package assets |
| Issues | Work entry and work tracking |
| PRs | Delivery and review |

## Cross-Repository Contracts

```text
Governance -> Operations: rules, evidence, approvals, promotion path
Knowledge -> Operations: similar work, playbooks, project memory, risks
Operations -> Runners: task slice, context pack, policy constraints, run ID
Runners -> Protocol: status events, artifacts, handoff payloads, errors
Adapters -> Operations: external requests -> internal IEF work objects
Governance -> Adapters: governance packs -> host-specific rules
Knowledge -> Governance / Adapters / Runners: context packs, patterns, decisions
```
