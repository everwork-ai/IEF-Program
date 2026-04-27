# ADR-0005: Use GitHub Project as Dynamic Command Center

**Status:** Accepted

**Date:** 2026-04-27

**Governance Profile:** Contract-Critical

## Context

IEF needs two different kinds of project status tracking:
1. Dynamic, real-time status (who is working on what, blockers, current phase)
2. Long-term, stable records (roadmap, decisions, design docs)

## Decision

Use an organization-level GitHub Project named **IEF Command Center** as the dynamic command center.

Use `IEF-Program` repository as the long-term source of truth for roadmap, RFCs, and ADRs.

## Responsibilities

| Type | Source of Truth | Examples |
|---|---|---|
| Dynamic status | IEF Command Center | Current phase, blockers, owner agent, priority changes |
| Long-term records | IEF-Program | Roadmap, RFCs, ADRs, operating model, agent rules |

## Required Fields

IEF Command Center must track:
- Status: Inbox / Ready / In Progress / Review / Blocked / Done / Archived
- Phase: P0-Setup / P1-Migration / P2-Core Loop / P3-Protocol / P4-Hardening
- Layer: Program / Governance / Knowledge / Operations / Protocol / Runners / Adapters
- Work Type: Design / Migration / Implementation / Review / Bug / Research / Decision
- Priority: P0 / P1 / P2 / P3
- Owner Agent: GPT / Claude / Codex / Qoder / OpenClaw / Human
- Risk: Low / Medium / High / Critical
- Review Required: No / Normal / Cross Review / Human Sign-off
- Target Repo: repository name
- Blocked By: issue or PR reference

## Consequences

**Positive:**
- Dynamic status does not pollute long-term documents
- GitHub-native integration with issues and PRs
- Real-time visibility for all stakeholders

**Negative:**
- Requires manual maintenance of project fields
- GitHub Projects v2 has limitations on auto-add for cross-repo issues

## Mitigation

Use manual add + GraphQL API for seed association where needed.

## Related

- ADR-0001: Use GitHub Project as Command Center (precursor)
- RFC-0002: IEF-Program Control Interface
