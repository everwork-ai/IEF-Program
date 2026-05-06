# IEF GitHub Workflow

This document defines the standard workflow for all IEF work: from issue creation to PR merge.

## Workflow Overview

```text
Issue created
  -> Issue added to IEF Command Center
  -> Execution plan commented
  -> Branch created
  -> Work performed
  -> PR opened
  -> PR linked to issue
  -> Review
  -> Merge
  -> Issue closed
```

## Agent Operating Mode

All IEF agents operate under the **GitHub-first control plane** defined in [`IEF_PROGRAM_CONTROLLER_MODE.md`](IEF_PROGRAM_CONTROLLER_MODE.md).

**Key rules:**
- GitHub issues, PRs, and comments are the durable source of truth. Chat history is ephemeral and must not be treated as authoritative.
- Local agents (Qoder, Codex App/IDE, Claude Code, Gemini CLI, Antigravity, OpenClaw/Hermes-style agents) must run in `SYNC_FROM_GITHUB` mode.
- Agents only modify files when a GitHub issue or PR comment explicitly requires action. Status/waiting/blocked comments do not authorize file changes.
- Agents must not self-merge, self-close issues, or modify sibling repos.

**SYNC_FROM_GITHUB startup sequence:**
1. Read target issue body and **comment stream** for the current repo.
2. Read related PR bodies and **comment streams**.
3. Read latest Program Controller / Program Agent comments on the program epic.
4. Read Codex review threads on the target PR.
5. Read PR body and changed files.
6. Read main branch docs / schemas / templates.

## Step 1 — Issue Creation

Every official task must start as a GitHub issue.

**Required fields:**
- Title with layer prefix: `[Program]`, `[Governance]`, `[Operations]`, etc.
- Clear description
- Acceptance criteria
- Governance profile declaration (Design-Lite / Contract-Critical / Implementation-Controlled)
- Target repository

**Issue templates:**
- `task.yml` — standard work item
- `epic.yml` — multi-step or cross-repo work
- `decision.yml` — architecture or policy decision
- `review.yml` — review or audit task

## Step 2 — IEF Command Center

After creation, the issue must be added to the **IEF Command Center** GitHub Project.

**Required project fields:**
- Status
- Phase
- Layer
- Work Type
- Priority
- Owner Agent
- Risk
- Review Required
- Target Repo

## Step 3 — Execution Plan

Before work begins, the assigned agent must comment an execution plan on the issue.

**Execution plan must include:**
- Steps to be taken
- Files to be created or modified
- Dependencies and blockers
- Governance profile confirmation
- Estimated effort

## Step 4 — Branch Creation

Create a feature branch from `main`.

**Branch naming:**
```text
issue-{number}-{short-description}
```

Example: `issue-4-bootstrap-program`

## Step 5 — Work and Commit

Perform the work on the feature branch.

**Commit message conventions:**
```text
type(scope): description

Types: docs, feat, fix, refactor, test, chore
```

Examples:
- `docs(operations): Add task lifecycle v0`
- `feat(protocol): Define AgentCardLite schema`
- `fix(runners): Correct runner interface boundary`

## Step 6 — PR Opening

Open a pull request when ready for review.

**PR must include:**
- Summary of changes
- Linked issue (use `Closes #123`, replacing `123` with the actual issue number)
- Files created/updated list
- Governance profile used
- Evidence (tests, dry-runs, screenshots)
- Risks
- Rollback plan

**PR template:** `.github/PULL_REQUEST_TEMPLATE.md`

## Step 7 — Review

**Review requirements by profile:**

| Profile | Review Requirement |
|---|---|
| Design-Lite | One reviewer check |
| Contract-Critical | Cross-review + Program or human approval |
| Implementation-Controlled | Review before merge |

**Reviewer checklist:**
- [ ] Issue is linked
- [ ] Acceptance criteria are addressed
- [ ] Layer boundaries are respected
- [ ] No unintended side effects
- [ ] Evidence is sufficient

## Step 8 — Merge and Close

After approval, merge the PR.

**Merge rules:**
- Use squash merge for single-logic changes
- Use regular merge for multi-commit stories
- Do not push directly to `main`

**After merge:**
- Issue should close automatically (if `Closes` keyword used)
- Update IEF Command Center status to Done
- Comment completion evidence on the issue

## Cross-Repository Work

For work that spans multiple repositories:

1. Create a parent epic issue in IEF-Program
2. Create child issues in each affected capability repo
3. Link child issues to parent epic
4. Add all issues to IEF Command Center
5. Open separate PRs per repository
6. Reference the epic in each PR

## Emergency Procedures

**Hotfix to main:**
1. Create issue documenting the emergency
2. Create branch from main
3. Make minimal fix
4. Fast-track review if critical
5. Merge and document post-mortem in issue

**Rollback:**
1. Revert the PR via GitHub
2. Comment on the original issue
3. Create follow-up issue for proper fix

## Untracked Work Policy

**Untracked work does not count as official IEF progress.**

If work cannot be linked to an issue in IEF Command Center, it is unofficial and must not be merged.
