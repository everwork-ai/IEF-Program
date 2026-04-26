# IEF Repository Matrix

| Repository | Layer | Codename | Core Responsibility | Does Not Own | Current Source |
|---|---|---|---|---|---|
| IEF-Program | Program | HQ | Project-family roadmap, RFCs, decision records, migration plan, GitHub operating model, AI-agent working rules. | Runtime governance rules, task state engine, runner implementations, protocol schemas. | New repository. |
| IEF-Governance | Governance | Charter | Rules, approvals, gates, review, audit, promotion, exceptions. | Project-family progress tracking, primary task state, runner implementation, long-term knowledge storage. | AgentSDLC upgrade. |
| IEF-Knowledge | Knowledge | Library | Session memory, project memory, decisions, playbooks, retrieval context. | Task scheduling, policy decisions, runner logic. | llm-knowledge-base upgrade. |
| IEF-Operations | Operations | Dispatch | Task, workflow, run state, queues, dependencies, retry, approval checkpoints, run ledger. | Program management, protocol standardization, concrete runner implementation, host injection. | New repository. |
| IEF-Protocol | Protocol | Relay | AgentCard, Capability, TaskEnvelope, Message, Artifact, StatusEvent, Handoff, schema validation. | Task state ownership, policy decisions, concrete transport runtime. | New repository. |
| IEF-Runners | Runners | Hands | Concrete runner implementations and shared runner interface. | System-wide governance, system-wide task state, host-specific instruction injection. | Absorb claude-worker. |
| IEF-Adapters | Adapters | Gateway | Host-surface mapping and IEF injection for Hermes, OpenClaw, Qoder, Codex, Claude Code, future gateways. | Core protocol design, global task state, knowledge storage semantics. | Harness-4-AIAgents upgrade. |
