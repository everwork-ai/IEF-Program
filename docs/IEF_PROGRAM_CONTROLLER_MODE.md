# IEF Program Controller Operating Mode

This document codifies the GitHub-first control plane for IEF Program execution.

## Principle

**GitHub is the single control surface and durable fact source for IEF Program execution.**

Chat history, session memory, and local agent state are ephemeral. They are not the source of truth. All durable instructions, status updates, blockers, and approvals must exist as GitHub issues, PRs, or comments.

## Roles

### Program Controller

- **Responsibility:** L1 control-plane coordination.
- **Authority:** Write issue comments, PR comments, review instructions, blocker/status comments, stale-thread judgments, `@codex review` triggers, human sign-off requests, and program issue coordination comments.
- **Limitation:** Must not directly modify capability repo files, directly fix bugs in capability repos, directly merge PRs, directly close core issues, or modify governance/protocol/operations substantive content without explicit Human Owner approval.

### Program Agent

- **Responsibility:** Cross-repo orchestration and dispatch.
- **Authority:** Read all program issues and PRs, report status, identify blockers, and request agent activation.
- **Limitation:** Does not directly implement capability repo work. Delegates to Repo Worker Agents.

### Repo Worker Agent

- **Responsibility:** Single-repo implementation and delivery.
- **Authority:** Modify files only when explicitly instructed by a GitHub issue or PR comment, create feature branches, open PRs, and reply to Codex review threads.
- **Limitation:** Must not modify files unless a GitHub issue or PR comment explicitly requires action. Must not modify sibling repos. Must not self-merge or self-close issues.

### Human Owner

- **Responsibility:** High-level authorization and sign-off.
- **Authority:** Approve operating mode changes, merge Contract-Critical PRs, close epics, and override Program Controller decisions.
- **Limitation:** Does not perform routine implementation work.

### Codex Reviewer

- **Responsibility:** Automated code and document review.
- **Authority:** Post review comments, approve PRs with reactions, and flag issues.
- **Limitation:** Does not modify files directly. Review output is advisory unless Program Controller or Human Owner elevates it to a blocker.

## Permission Boundaries

| Level | Scope | Who | Examples |
|---|---|---|---|
| **L1** | Control-plane writes | Program Controller | Issue comments, PR comments, status updates, `@codex review` triggers, blocker declarations |
| **L2** | Cross-repo reads and coordination | Program Agent | Read all repo issues/PRs, report cross-repo status, dispatch work to Repo Worker Agents |
| **L3** | Single-repo implementation | Repo Worker Agent | File edits, branch creation, PR opening, Codex thread replies, test execution |

**Rule:** No agent may operate above its authorized level without explicit Human Owner approval.

## SYNC_FROM_GITHUB Mode

Local agents (Qoder, Codex App/IDE, Claude Code, Gemini CLI, Antigravity, OpenClaw/Hermes-style agents) must run in `SYNC_FROM_GITHUB` mode.

### Startup Sequence

1. **Read target issue** — Identify the issue assigned to the current repo.
2. **Read related PRs** — Identify open PRs linked to the target issue.
3. **Read Program Controller comments** — Read the latest comments on IEF-Program#6 (or current program epic) for directives.
4. **Read Codex review threads** — Check for unresolved review comments on the target PR.
5. **Read PR body and changed files** — Understand the current state of delivery.
6. **Read main branch docs/schemas/templates** — Understand existing conventions before modifying files.

### Execution Rules

1. **Only act when explicitly required.** If the latest GitHub comment is status/waiting/blocked, do not modify files.
2. **Use feature branches.** Never push directly to `main`.
3. **Only modify the current repo.** Do not touch sibling repos.
4. **Only modify files explicitly requested.** Do not perform speculative refactoring.
5. **Push and report.** After every modification, push to the PR branch and post a summary comment on the PR.
6. **Reply to Codex findings.** If Codex posted review threads, reply to each one.
7. **Trigger Codex review.** Comment `@codex review` after pushing changes.
8. **Do not self-merge.** Wait for Program Controller or Human Owner.
9. **Do not self-close issues.** Wait for Program Controller or Human Owner.
10. **Do not start downstream work.** If a task requires another repo, report a blocker on the PR.

### Agent-Specific Consumption Patterns

| Agent | How It Consumes GitHub Control Plane |
|---|---|
| **Qoder CLI** | Reads issues/PRs via `gh` CLI. Responds to `SYNC_FROM_GITHUB` command. Pushes changes and comments via `gh`. |
| **Codex App/IDE** | Reads PRs via GitHub integration. Responds to `@codex review` and `@codex address that feedback`. |
| **Claude Code** | Reads issues/PRs via GitHub MCP or `gh` CLI. Can be instructed to sync from specific issue URLs. |
| **Gemini CLI** | Reads issues/PRs via GitHub API or `gh` CLI. Responds to explicit issue references in prompts. |
| **Antigravity** | Reads issues/PRs via GitHub integration. Consumes issue body and acceptance criteria as task specification. |
| **OpenClaw / Hermes-style** | Reads issues/PRs via GitHub API. Uses issue comments as instruction stream and PR comments as delivery confirmation. |

## Source of Truth Hierarchy

| Priority | Source | Ephemeral? |
|---|---|---|
| 1 | GitHub issues | Durable |
| 2 | GitHub PRs | Durable |
| 3 | GitHub issue/PR comments | Durable |
| 4 | Merged control documents (RFCs, ADRs, docs) | Durable |
| 5 | Chat history | Ephemeral |
| 6 | Local agent memory | Ephemeral |

**Rule:** If there is a conflict between a GitHub comment and chat history, the GitHub comment wins.

## Reporting Format

Repo Worker Agents should produce a sync report after every `SYNC_FROM_GITHUB` cycle:

```markdown
Repo Worker Sync Report

- Repo: <owner/repo>
- Target issue: #<number> — <title>
- Target PR: #<number> — <title> (or "None")
- Latest head: <commit-sha>
- Current status: <status summary>
- Action required: <yes/no — with justification>
- Files allowed to modify: <list or "None">
- Files modified: <list or "None">
- Codex status: <clean / findings pending / blocked>
- Blockers: <list or "None">
- Next required owner: <Program Controller / Human Owner / Repo Worker>
```

## Transition from Chat-First

The previous operating mode relied on copying prompts through chat. This is deprecated because:

- Chat history is not searchable by other agents.
- Chat history is not versioned.
- Chat history does not trigger GitHub notifications.
- Chat history cannot be referenced in cross-repo work.

All agents must transition to consuming GitHub as the primary instruction surface.
