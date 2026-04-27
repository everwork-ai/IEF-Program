# RFC-0003: IEF Repository Matrix

**Status:** Proposed

**Governance Profile:** Contract-Critical

## Problem

IEF has seven repositories with overlapping concerns. Without an explicit matrix:
- Agents implement the wrong responsibilities in the wrong repos
- Cross-repo contracts are violated
- Governance rules are confused with project control
- Protocol objects are invented locally instead of centralized

## Proposal

Adopt a fixed repository matrix with layer, codename, responsibility, boundary, and source mapping.

### Matrix

| Repository | Layer | Codename | Core Responsibility | Does Not Own | Current Source |
|---|---|---|---|---|---|
| IEF-Program | Program | HQ | Project-family roadmap, RFCs, decision records, migration plan, GitHub operating model, AI-agent working rules. | Runtime governance rules, task state engine, runner implementations, protocol schemas. | New repository. |
| IEF-Governance | Governance | Charter | Rules, approvals, gates, review, audit, promotion, exceptions. | Project-family progress tracking, primary task state, runner implementation, long-term knowledge storage. | AgentSDLC upgrade. |
| IEF-Knowledge | Knowledge | Library | Session memory, project memory, decisions, playbooks, retrieval context. | Task scheduling, policy decisions, runner logic. | llm-knowledge-base upgrade. |
| IEF-Operations | Operations | Dispatch | Task, workflow, run state, queues, dependencies, retry, approval checkpoints, run ledger. | Program management, protocol standardization, concrete runner implementation, host injection. | New repository. |
| IEF-Protocol | Protocol | Relay | AgentCard, Capability, TaskEnvelope, Message, Artifact, StatusEvent, Handoff, schema validation. | Task state ownership, policy decisions, concrete transport runtime. | New repository. |
| IEF-Runners | Runners | Hands | Concrete runner implementations and shared runner interface. | System-wide governance, system-wide task state, host-specific instruction injection. | Absorb claude-worker. |
| IEF-Adapters | Adapters | Gateway | Host-surface mapping and IEF injection for Hermes, OpenClaw, Qoder, Codex, Claude Code, future gateways. | Core protocol design, global task state, knowledge storage semantics. | Harness-4-AIAgents upgrade. |

### Cross-Repo Contracts

```text
Governance -> Operations: rules, evidence, approvals, promotion path
Knowledge -> Operations: similar work, playbooks, project memory, risks
Operations -> Runners: task slice, context pack, policy constraints, run ID
Runners -> Protocol: status events, artifacts, handoff payloads, errors
Adapters -> Operations: external requests -> internal IEF work objects
Governance -> Adapters: governance packs -> host-specific rules
Knowledge -> Governance / Adapters / Runners: context packs, patterns, decisions
```

### Anti-Patterns

1. **Governance-as-PMO** — IEF-Governance must not track project progress
2. **Chat-as-state** — Do not use conversation as authoritative work state
3. **Local protocol invention** — Do not create object formats in downstream repos before IEF-Protocol drafts exist
4. **Runner lock-in** — Do not treat ClaudeCode as the architecture
5. **Adapter confusion** — Do not mistake host prompt injection for core protocol

## Alternatives Considered

| Alternative | Rejected Because |
|---|---|
| Merge Program and Governance | Violates separation of rules and coordination |
| Merge Protocol and Operations | Protocol should stabilize independently of task state logic |
| Merge Runners and Adapters | Runners execute; Adapters bind to hosts. Different concerns. |
| Add more repos (Identity, Evals, Runtime) | Premature. Prove core loop first. |

## Impact

- All agents must respect repo boundaries
- All contracts must reference the matrix
- New repos require ADR approval

## Migration

Existing repos map as follows:
- `AgentSDLC` -> `IEF-Governance`
- `llm-knowledge-base` -> `IEF-Knowledge`
- `Harness-4-AIAgents` -> `IEF-Adapters`
- `claude-worker` -> `IEF-Runners/runners/claudecode`
