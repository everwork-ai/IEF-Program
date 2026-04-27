# ADR-0004: Use IEF-Program as Control Interface

**Status:** Accepted

**Date:** 2026-04-27

**Governance Profile:** Contract-Critical

## Context

IEF has six capability repositories (Governance, Knowledge, Operations, Protocol, Runners, Adapters). Without a dedicated control interface:
- Work enters repositories independently
- Cross-repo dependencies are invisible
- Agent behavior is inconsistent
- Governance rules and project coordination are confused

## Decision

Use `IEF-Program` as the dedicated HQ / program control interface.

## Responsibilities

- Project-family roadmap
- RFCs and architecture design records
- GitHub operating model and workflow definitions
- AI agent working rules
- Cross-repository contract issues and coordination
- Program status reports and definition of done

## Boundaries

IEF-Program does **not** own:
- Runtime governance rules (IEF-Governance)
- Task state engine (IEF-Operations)
- Protocol schemas (IEF-Protocol)
- Runner implementations (IEF-Runners)
- Host adapter code (IEF-Adapters)
- Knowledge storage (IEF-Knowledge)

## Consequences

**Positive:**
- Clear separation between project coordination and capability implementation
- Single entry point for AI agents
- Long-term decisions are recorded and discoverable
- IEF-Governance is freed to focus on governance rules only

**Negative:**
- Adds one more repository to maintain
- Requires agents to check IEF-Program before starting work

## Alternatives Considered

| Alternative | Rejected Because |
|---|---|
| Use IEF-Governance as control center | Governance defines rules; it should not control project progress |
| Use IEF Command Center as sole control | GitHub Project is dynamic status only, not long-term decision storage |
| Spread control docs across all repos | Creates fragmentation; no single entry point |

## Related

- RFC-0002: IEF-Program Control Interface
- ADR-0001: Use GitHub Project as Command Center
