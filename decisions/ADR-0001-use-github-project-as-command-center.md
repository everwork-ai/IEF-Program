# ADR-0001: Use GitHub Project as IEF Command Center

## Status

Accepted

## Context

IEF is a multi-repository, multi-agent engineering program. Multiple AI agents may work concurrently. We need a single dynamic control surface for cross-repository progress tracking.

## Decision

Use an organization-level GitHub Project named **IEF Command Center** as the dynamic project control surface.

## Rationale

- Repositories hold assets.
- Issues hold work items.
- PRs hold deliveries.
- GitHub Project provides cross-repo status, filters, fields, and views without creating a custom dashboard.

## Consequence

- IEF-Program stores long-term facts (roadmap, RFCs, ADRs); it does not manually maintain real-time task status.
- IEF-Governance stores governance rules only; it does not own project-family progress tracking.
- All agents must ensure their issues are added to IEF Command Center.
