# IEF Agent Working Rules

## Task Entry

1. Only work from GitHub Issues.
2. Every official issue must be added to IEF Command Center.
3. Every issue must declare Layer, Target Repo, Owner Agent, Acceptance Criteria, and Review Requirement.
4. Complex issues must be split into sub-issues or linked issues.

## Execution

1. Before work begins, comment an execution plan on the issue.
2. During work, comment key findings and blockers.
3. Do not modify unrelated repositories without linked issues.
4. Do not push directly to main.
5. Create a branch and PR for delivery.

## Delivery

1. PR must link the issue using `Closes` / `Fixes` / `Resolves`.
2. PR must include summary, evidence, risks, rollback plan, and reviewer notes.
3. High-risk work requires review before merge.
4. After merge, the issue should close automatically or be manually closed by reviewer.

## Project Visibility

1. Every active issue must appear in IEF Command Center.
2. Every PR should reference a Project-tracked issue.
3. Every status change should happen in issue/PR/Project, not private chat only.

## Untracked Work Policy

**Untracked work does not count as official IEF progress.**

Agents must not perform untracked cross-repository modifications. If an agent cannot link its work to an issue in IEF Command Center, the work is unofficial.
