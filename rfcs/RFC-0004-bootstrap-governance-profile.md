# RFC-0004: Bootstrap Governance Profiles

**Status:** Proposed

**Governance Profile:** Contract-Critical

## Problem

IEF needs governance from day one, but a full governance system would slow down bootstrap. We need lightweight profiles that:
- Prevent untracked work
- Ensure contract quality
- Allow fast iteration for low-risk changes
- Require rigorous review for critical contracts

## Proposal

Define three bootstrap governance profiles:

| Profile | Use For | Requirements |
|---|---|---|
| Design-Lite | README/doc/stub updates | linked issue, acceptance criteria, PR, reviewer check |
| Contract-Critical | Protocol objects, lifecycle, interfaces, contracts | design doc, explicit I/O, explicit boundary, schema or example, cross-review, Program approval |
| Implementation-Controlled | Code, scripts, validators, runner logic | linked issue, design reference, PR, test or dry-run evidence, rollback plan, review before merge |

### Design-Lite

**Purpose:** Low-risk documentation and structural changes.

**Requirements:**
1. Linked issue exists
2. Acceptance criteria stated in issue
3. PR opened
4. Reviewer check
5. No broken links or obvious errors

**Approval:** One reviewer.

**Examples:**
- Fix typo in README
- Add doc comment
- Reorganize folder structure
- Update status report

### Contract-Critical

**Purpose:** Objects, interfaces, and contracts that downstream repos depend on.

**Requirements:**
1. Linked issue exists
2. Design document or RFC reference
3. Explicit input/output contract documented
4. Explicit boundary documented (owns / does not own)
5. Schema, example, or validation rule provided
6. Cross-review from at least one other repo perspective
7. Program Controller or human approval before merge
8. Rollback plan documented if replacing existing contract

**Approval:** Cross-review + Program Controller or human sign-off.

**Examples:**
- Define JSON schema for TaskEnvelope
- Define task lifecycle state machine
- Define runner interface
- Define adapter contract
- Define governance profile (meta)

### Implementation-Controlled

**Purpose:** Code and automation that affects runtime behavior.

**Requirements:**
1. Linked issue exists
2. Design reference (RFC, ADR, or contract doc)
3. PR opened
4. Test or dry-run evidence attached to PR
5. Rollback plan documented
6. Review before merge
7. No regression in existing capabilities

**Approval:** Review before merge. Human or Program Controller approval for high-risk changes.

**Examples:**
- Implement runner logic
- Add validation script
- Add GitHub Action
- Modify adapter code
- Add state machine implementation

## Mandatory Checks for All Profiles

1. **Issue link** — Every PR must reference an issue.
2. **IEF Command Center** — Every issue should be visible in the project board.
3. **No direct main push** — All delivery via PR.
4. **Layer boundary respect** — Do not implement another layer's responsibility.

## Escalation

If the correct profile is unclear, default to **Contract-Critical** and ask in the issue or IEF Command Center.

## Future Evolution

These are bootstrap profiles. As IEF matures:
- Profiles may be refined or split
- Automated checks may replace manual reviewer steps
- Risk classification may become more granular
- Evaluation gates may be added

Any change to profiles requires Contract-Critical review.
