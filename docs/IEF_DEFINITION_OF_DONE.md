# IEF Definition of Done

This document defines the acceptance criteria for each IEF governance profile.

All IEF work must declare which governance profile it uses. The profile determines what "done" means.

## Governance Profiles

### Design-Lite

For README updates, documentation stubs, non-normative comments, and structural reorganization that does not change runtime behavior.

**Definition of Done:**
- [ ] Linked issue exists
- [ ] Acceptance criteria are stated in the issue
- [ ] Pull request is opened
- [ ] Reviewer has checked the change
- [ ] No broken links or obvious errors

**Approval:** One reviewer check.

### Contract-Critical

For protocol objects, task lifecycle definitions, runner interfaces, adapter contracts, knowledge context formats, and governance profiles.

**Definition of Done:**
- [ ] Linked issue exists
- [ ] Design document or RFC reference
- [ ] Explicit input/output contract documented
- [ ] Explicit boundary documented (what it owns / does not own)
- [ ] Schema, example, or validation rule provided
- [ ] Cross-review from at least one other repo perspective
- [ ] Program Controller or human approval before merge
- [ ] Rollback plan documented if replacing existing contract

**Approval:** Cross-review + Program Controller or human sign-off.

### Implementation-Controlled

For code, scripts, validators, runner logic, GitHub automation, adapter code, and any change that affects runtime behavior.

**Definition of Done:**
- [ ] Linked issue exists
- [ ] Design reference (RFC, ADR, or contract doc)
- [ ] Pull request is opened
- [ ] Test or dry-run evidence attached to PR
- [ ] Rollback plan documented
- [ ] Review before merge
- [ ] No regression in existing capabilities

**Approval:** Review before merge. Human or Program Controller approval for high-risk changes.

## Mandatory Checks for All Profiles

1. **Issue link** — Every PR must reference an issue.
2. **IEF Command Center** — Every issue should be visible in the project board.
3. **No direct main push** — All delivery via PR.
4. **Layer boundary respect** — Do not implement another layer's responsibility.

## Profile Selection Guide

| Change Type | Profile |
|---|---|
| Fix typo in README | Design-Lite |
| Add doc comment | Design-Lite |
| Reorganize folder structure | Design-Lite |
| Define JSON schema | Contract-Critical |
| Define task lifecycle | Contract-Critical |
| Define runner interface | Contract-Critical |
| Add governance profile | Contract-Critical |
| Implement runner logic | Implementation-Controlled |
| Add validation script | Implementation-Controlled |
| Add GitHub Action | Implementation-Controlled |
| Modify adapter code | Implementation-Controlled |

## Escalation

If the correct profile is unclear, default to **Contract-Critical** and ask in the issue or IEF Command Center.
