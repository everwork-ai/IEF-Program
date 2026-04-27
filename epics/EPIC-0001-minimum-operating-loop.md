# EPIC-0001: Establish IEF v0 Minimum Operating Loop

**Epic Title:** [Program] Establish IEF v0 Minimum Operating Loop

**Status:** Proposed

**Governance Profile:** Contract-Critical

## Goal

Prove that IEF can manage one AI employee task from creation to completion:

```text
task created -> governed -> assigned -> executed -> artifact produced -> reviewed -> summarized -> closed
```

## Success Criteria

- [ ] A task can be created through IEF-Program
- [ ] A governance profile can be selected from IEF-Governance
- [ ] The task object follows IEF-Protocol contracts
- [ ] The task/run lifecycle follows IEF-Operations state machine
- [ ] Execution is performed by an IEF-Runners runner
- [ ] The integration entry is defined by IEF-Adapters
- [ ] Run summary and context are captured by IEF-Knowledge
- [ ] The task closes through a PR linked to an issue

## Scope

### In Scope

- One end-to-end task execution
- Basic governance profile selection
- Protocol object stubs (AgentCardLite, TaskEnvelope, RunEvent, ArtifactRef)
- Task lifecycle v0 (create, assign, run, complete)
- One runner execution (ClaudeCode)
- One adapter entry point
- Knowledge capture (run summary, context pack)

### Out of Scope

- Multi-agent handoff
- Complex workflow orchestration
- Advanced approval chains
- External protocol gateways
- Runtime service decomposition
- Identity and authorization

## Dependency Order

```text
IEF-Program Epic (this document)
├── IEF-Governance Bootstrap Profiles
│   └── Design-Lite / Contract-Critical / Implementation-Controlled
├── IEF-Protocol Object Model
│   ├── AgentCardLite
│   ├── TaskEnvelope
│   ├── RunEvent
│   ├── ArtifactRef
│   └── ContextRef
├── IEF-Operations Lifecycle
│   ├── Task lifecycle v0
│   ├── Run ledger v0
│   └── Approval checkpoint v0
├── IEF-Runners Interface
│   ├── Runner interface v0
│   └── ClaudeCode runner boundary
├── IEF-Knowledge Context Pack
│   ├── Context pack format v0
│   └── Run summary format v0
└── IEF-Adapters Contract
    ├── Host adapter contract v0
    └── One adapter stub
```

## Child Issues

| Issue | Repository | Description | Status |
|---|---|---|---|
| [IEF-Governance#2](https://github.com/everwork-ai/IEF-Governance/issues/2) | IEF-Governance | Define bootstrap governance profiles for IEF v0 | Created / Ready for contract draft |
| [IEF-Protocol#2](https://github.com/everwork-ai/IEF-Protocol/issues/2) | IEF-Protocol | Define IEF v0 shared object model | Created / Ready for contract draft |
| [IEF-Operations#2](https://github.com/everwork-ai/IEF-Operations/issues/2) | IEF-Operations | Define IEF v0 task and run lifecycle | Created / Ready for contract draft |
| [IEF-Runners#2](https://github.com/everwork-ai/IEF-Runners/issues/2) | IEF-Runners | Define runner interface v0 and ClaudeCode runner boundary | Created / Ready for contract draft |
| [IEF-Knowledge#2](https://github.com/everwork-ai/IEF-Knowledge/issues/2) | IEF-Knowledge | Define context pack and run summary v0 | Created / Ready for contract draft |
| [IEF-Adapters#2](https://github.com/everwork-ai/IEF-Adapters/issues/2) | IEF-Adapters | Define host adapter contract v0 | Created / Ready for contract draft |

## Governance

This epic uses the **Contract-Critical** governance profile:
- Design doc reference required
- Explicit input/output contracts
- Explicit boundary definitions
- Cross-review from multiple repo perspectives
- Program Controller approval before merge

## Timeline

| Phase | Target | Deliverable |
|---|---|---|
| Phase 1 | Week 1 | Governance profiles + Protocol object model |
| Phase 2 | Week 2 | Operations lifecycle + Runners interface |
| Phase 3 | Week 3 | Knowledge context pack + Adapters contract |
| Phase 4 | Week 4 | First end-to-end task execution |

## Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Protocol/Operations drafts delayed | Blocks all downstream work | Start with minimal viable contracts |
| Runner interface mismatch | Breaks execution loop | Align with ClaudeCode runner early |
| Governance too heavy | Slows iteration | Use lightweight bootstrap profiles |
| Knowledge format undefined | Loses run memory | Define context pack early |

## Rollback Plan

If the minimum operating loop cannot be established:
1. Document blockers in this epic
2. Create follow-up issues for each blocker
3. Re-scope to smaller proof-of-concept
4. Do not proceed to Phase 2 scaling until loop is proven
