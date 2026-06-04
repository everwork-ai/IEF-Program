# IEF Program Controller Operating Mode

This document codifies the GitHub-first control plane for IEF Program execution.

## Principle

**GitHub is the single control surface and durable fact source for IEF Program execution.**

Chat history, session memory, and local agent state are ephemeral. They are not the source of truth. All durable instructions, status updates, blockers, and approvals must exist as GitHub issues, PRs, or comments.

## Roles

### Program Controller

- **Responsibility:** L1 control-plane coordination.
- **Authority:** Write issue comments, PR comments, review instructions, blocker/status comments, stale-thread judgments, `@codex review` triggers, human sign-off requests, and program issue coordination comments.
- **Limitation:** By default, must not directly modify capability repo files, directly fix bugs in capability repos, directly merge PRs, directly close core issues, or modify governance/protocol/operations substantive content.
  - **Standing delegation exception:** Human Owner has granted conditional L3 authority under the standing delegation, but **only when** a GitHub issue or PR directive defines scope, target files, and expected output. For governance/protocol/operations core contracts, human sign-off is still required before merge. Without an explicit directive, capability repo edits remain prohibited.

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

## Role Operating Boundaries

These boundaries describe what each role is responsible for. They are independent of the numeric L0-L4 delegation levels defined in Human Owner Standing Delegation.

| Role | Scope | Examples |
|---|---|---|
| **Program Controller** | Control-plane writes | Issue comments, PR comments, status updates, `@codex review` triggers, blocker declarations |
| **Program Agent** | Cross-repo reads and coordination | Read all repo issues/PRs, report cross-repo status, dispatch work to Repo Worker Agents |
| **Repo Worker Agent** | Single-repo implementation | File edits, branch creation, PR opening, Codex thread replies, test execution |

**Rule:** No agent may operate outside its role boundary or delegated authority without explicit Human Owner approval.

## Human Owner Standing Delegation

Human Owner may grant standing delegation to the Program Controller. This delegation is recorded as a durable GitHub comment and remains active until revoked or superseded by a later Human Owner comment.

**Platform limitation:** Platform and tool-level confirmation prompts may still appear and cannot be bypassed by this delegation. The delegation governs IEF organizational authority, not local security controls.

### Delegated levels

| Level | Scope | Default | Conditions |
|---|---|---|---|
| **L0** | Read across all everwork-ai IEF repositories | Allowed | No additional authorization required. |
| **L1** | Control-plane writes | Allowed by default | Issue comments, PR comments, review instructions, blocker/status comments, stale-thread judgments, `@codex review` triggers, human sign-off requests, downstream unblock/block directives. |
| **L2** | IEF-Program documentation changes | Allowed via branch + PR | Program Controller may create and update PRs, but must not merge without human sign-off unless explicitly authorized. |
| **L3** | Capability repo content changes | Allowed only with explicit directive | Requires a GitHub issue or PR directive defining scope, target files, and expected output. Program Controller may create branches, push commits, open/update PRs, and reply to review threads. Must not directly merge or close core issues without human approval. |
| **L4** | Merge / close actions | Require explicit sign-off | Human Owner sign-off required unless the PR falls under a separately approved auto-merge rule. |

### Prohibited actions

Even under standing delegation, the Program Controller must not:

- Delete repositories
- Change organization permissions
- Change secrets, tokens, or billing settings
- Force-push
- Change branch protection rules
- Perform production deployment
- Merge governance, protocol, or operations core contracts without human sign-off

### Revocation

Human Owner may revoke or supersede this delegation at any time by posting a new comment on the active program epic or the PR where the delegation was recorded.

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

- Fetches the latest issue body, **issue comments**, PR body, **PR comments**, and Codex review threads from GitHub.
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
| `READY_FOR_PROGRAM_REVIEW` | Agent has delivered; report delivered status and wait for Program Controller review. |
| `READY_FOR_HUMAN_SIGNOFF` | Program Controller approves; report waiting-human status and wait for Human Owner sign-off. |
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

| Priority | Source | Applies To |
|---|---|---|
| 1 | Merged RFCs / ADRs / control docs | Standing policy, architecture, contracts, governance, Protocol schemas, durable decisions |
| 2 | Latest authorized GitHub issue/PR comments | Current operational directives within active issue/PR only |
| 3 | Issue body / PR body | Baseline scope and delivery context |
| 4 | Chat history / local memory | Non-authoritative unless reflected in GitHub |

**Rules:**
- Merged RFCs, ADRs, and control documents are authoritative for standing policy. Comments must not override merged contracts, governance profiles, Protocol schemas, or ADR/RFC decisions.
- Latest authorized GitHub comments override stale issue/PR bodies only for current operational directives within an active issue or PR.
- If there is a conflict between a GitHub comment and chat history, the GitHub comment wins.
- If a comment needs to change standing policy, it must create or update a PR against the relevant durable document.

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
