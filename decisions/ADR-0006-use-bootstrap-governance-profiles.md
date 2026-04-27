# ADR-0006: Use Bootstrap Governance Profiles

**Status:** Accepted

**Date:** 2026-04-27

**Governance Profile:** Contract-Critical

## Context

IEF needs governance from day one, but a full governance system would slow down bootstrap. We need a lightweight system that:
- Scales with risk
- Prevents untracked work
- Does not require heavy process for trivial changes
- Requires rigor for contracts that downstream repos depend on

## Decision

Adopt three bootstrap governance profiles:

| Profile | Use For | Requirements |
|---|---|---|
| Design-Lite | README/doc/stub updates | linked issue, acceptance criteria, PR, reviewer check |
| Contract-Critical | Protocol objects, lifecycle, interfaces, contracts | design doc, explicit I/O, explicit boundary, schema or example, cross-review, Program approval |
| Implementation-Controlled | Code, scripts, validators, runner logic | linked issue, design reference, PR, test or dry-run evidence, rollback plan, review before merge |

## Rationale

**Why three profiles?**
- One profile is too rigid; it forces heavy process on trivial changes
- Two profiles (light/heavy) loses the distinction between "contract definition" and "code implementation"
- Three profiles capture the natural risk gradient: docs -> contracts -> code

**Why not more?**
- Too many profiles create confusion
- Bootstrap phase should be simple
- Future evolution can add granularity

## Consequences

**Positive:**
- Fast iteration for docs and stubs
- Rigorous review for contracts
- Evidence-based delivery for code
- Clear escalation path

**Negative:**
- Agents must choose the right profile
- Wrong profile choice can delay work

## Mitigation

- Default to Contract-Critical if unclear
- Document examples for each profile
- Allow profile change during review

## Future Evolution

These are bootstrap profiles. As IEF matures:
- Automated checks may replace manual steps
- Risk classification may become more granular
- Evaluation gates may be added

Any change to profiles requires Contract-Critical review.

## Related

- RFC-0004: Bootstrap Governance Profiles
- ADR-0004: Use IEF-Program as Control Interface
