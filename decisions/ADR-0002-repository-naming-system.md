# ADR-0002: Repository Naming System

## Status

Accepted

## Context

Multiple naming iterations occurred (Work/Workers, Orchestration/Orchestrationers, Execution). We need a stable, final naming system.

## Decision

Use these repository names:

- `IEF-Program`
- `IEF-Governance`
- `IEF-Knowledge`
- `IEF-Operations`
- `IEF-Protocol`
- `IEF-Runners`
- `IEF-Adapters`

Use these internal codenames:

- Program = HQ
- Governance = Charter
- Knowledge = Library
- Operations = Dispatch
- Protocol = Relay
- Runners = Hands
- Adapters = Gateway

## Rationale

- Repository names remain direct and executable.
- Codenames provide narrative but do not replace clear repo names.
- `Operations` is broader than workflow/task management.
- `Runners` clearly indicates concrete execution implementations.
- `Forge` is rejected because it duplicates the Foundry meaning of IEF.

## Consequence

Deprecated names must not be reused: `IEF-Work`, `IEF-Workers`, `IEF-Orchestration`, `IEF-Execution`, `IEF-Forge`.
