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

## Program Controller Execution Contract

The Program Controller must adhere to the following rules when directing agents.

### Write-before-act rule

- Program Controller must write durable instructions into GitHub **before** expecting agents to act.
- Program Controller must **not** assume agents saw chat messages.

### Directive placement rule

- Repo-specific work needs a directive on the **target repo issue or PR**.
- Cross-repo sequencing should also be recorded on the **Program issue or epic**.
- **Program-level status updates alone are not enough to activate a repo worker.** Repo-specific work must have an explicit directive on the target repo issue or PR. If a directive appears only in `IEF-Program`, repo workers may treat it as context but not as authorization to modify files.

### Directive labels

Program Controller directives should use explicit labels so agents can classify them automatically:

| Label | Meaning |
|---|---|
| `ACTION REQUIRED` | Agent must perform work now. |
| `BLOCKED` | Agent must stop and report why. |
| `WAITING_HUMAN` | Agent must pause until Human Owner responds. |
| `WAITING_CODEX` | Agent must pause until Codex review completes. |
| `READY_FOR_PROGRAM_REVIEW` | Agent has delivered; Program Controller should review. |
| `READY_FOR_HUMAN_SIGNOFF` | Program Controller approves; Human Owner should sign off. |
| `UNBLOCKED` | Previous blocker is resolved; agent may resume. |

### Directive completeness

A directive must include enough context for the agent to act **without chat history**:

- Target repo
- Target issue or PR
- Allowed scope (what the agent may touch)
- Expected output (what "done" looks like)
- PR state rule (keep draft / do not merge / do not close issue)

## Standard Directive Template

Program Controller comments should follow this structure when issuing actionable directives:

```markdown
## Directive

- **Target repo:** owner/repo
- **Target issue:** #N
- **Target PR:** #M (or "None — create one")
- **Branch:** issue-N-description
- **Context:** <why this work is needed>
- **Allowed scope:** <files or areas the agent may modify>
- **Files in scope:** <explicit file list if known>
- **Expected output:** <what the agent must deliver>
- **State rule:** <keep draft / do not merge / do not close issue / ...>
- **Label:** ACTION REQUIRED / BLOCKED / WAITING_HUMAN / ...
```

## Controller-Agent Interaction Loop

The normal execution loop is:

1. **Program Controller writes directive** to GitHub (issue or PR comment).
2. **Agent runs `SYNC_FROM_GITHUB`.**
3. **Agent reads latest authorized directives.**
4. **Agent determines whether action is required.** If the latest directive is status-only, the agent reports status and stops.
5. **Agent acts only within the stated scope.**
6. **Agent reports back** on the issue or PR.
7. **Program Controller reviews the report.**
8. **Program Controller decides next state:** request changes, trigger Codex review, request human sign-off, or unblock the next repo.

## SYNC_FROM_GITHUB Semantics

`SYNC_FROM_GITHUB` is a **pull-based sync command**. It does not automatically authorize file changes.

### What it does

- Fetches the latest issue body, PR body, and comments from GitHub.
- Rebuilds the agent's local understanding of the current state.

### What it does not do

- It does **not** bypass the requirement for an explicit `ACTION REQUIRED` directive.
- It does **not** authorize the agent to modify files just because the sync succeeded.

### Decision rule after sync

| Latest directive | Agent action |
|---|---|
| `ACTION REQUIRED` | Execute within stated scope, then report. |
| `BLOCKED` | Report blocker, stop. |
| `WAITING_HUMAN` / `WAITING_CODEX` | Report waiting status, stop. |
| `UNBLOCKED` | Resume previously blocked work if scope is still valid. |
| No labeled directive | Report status only, do not modify files. |

### Stale body override rule

**Latest authorized GitHub control comments override stale issue or PR body text** for current operational directives. If an issue body says one thing but a recent comment says another, the comment wins.

## Program Controller Pre-Flight Checklist

Before telling Human Owner to start or re-run an agent, Program Controller should confirm:

- [ ] The directive exists on the **target issue or PR**.
- [ ] The directive includes **scope and expected output**.
- [ ] The directive states **PR state rules**.
- [ ] The directive is **discoverable by `SYNC_FROM_GITHUB`** (i.e., it is a public comment, not chat-only).

## Reporting Rules

### Before a PR exists

If no PR has been opened yet, the agent must report progress and status **on the issue**.

### After a PR exists

Once a PR is opened, the agent must report **on the PR**. The issue may still receive high-level status updates, but detailed delivery summaries belong on the PR.

### Transition rule

When the agent creates a PR, it should post a final summary comment on the issue linking to the new PR, then continue reporting on the PR.

## Agent-Specific Consumption Patterns

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
| 1 | GitHub issue/PR comments (latest authorized) | Durable |
| 2 | GitHub issues | Durable |
| 3 | GitHub PRs | Durable |
| 4 | Merged control documents (RFCs, ADRs, docs) | Durable |
| 5 | Chat history | Ephemeral |
| 6 | Local agent memory | Ephemeral |

**Rule:** If there is a conflict between a GitHub comment and chat history, the GitHub comment wins. If there is a conflict between a recent comment and an older issue body, the recent comment wins.

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

## Relationship to IEF-Protocol

This document defines the **human-readable Program coordination protocol**. It is not the same as `IEF-Protocol` JSON schemas. If this interaction model later needs machine-readable messages, `IEF-Protocol` should define those objects in a separate Contract-Critical task.
