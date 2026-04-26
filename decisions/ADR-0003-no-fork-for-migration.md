# ADR-0003: No Fork for Migration

## Status

Accepted

## Context

Existing repositories need to move into the IEF family under `everwork-ai`.

## Decision

Use **transfer + rename** for existing repositories. Do not fork.

## Rationale

Forks imply upstream/downstream contribution relationships. IEF migration is a project-family re-architecture, not a fork-and-contribute workflow. Transfer preserves history, issues, PRs, stars, and followers.

## Consequence

- Old repository names will redirect to new names.
- No duplicate repositories for the same layer.
- `claude-worker` becomes `IEF-Runners` after transfer+rename, then restructured internally.
