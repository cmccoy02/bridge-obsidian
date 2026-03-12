# Tech Debt - TDR Template

Parent: [[tech debt]]

## Template

```md
# TDR-<id>: <short title>

- Status: Open | In Progress | Closed | Accepted
- Owner:
- Component:
- Debt Type:
- Secondary Type:
- Origin Date:
- Last Review:
- Next Review:

## Context
What decision or constraint created this debt?

## Current State
Describe the technical condition and symptoms.

## Principal Estimate
Engineering effort to remediate now.

## Interest Estimate
Recurring cost per sprint/month if not fixed.

## Risk
Reliability, security, compliance, or business impact.

## Evidence
Metrics, incidents, hotspots, test gaps, affected users.

## Target State
What "good" looks like after remediation.

## Remediation Plan
Milestones, dependencies, and rollback strategy.

## Success Metrics
How we verify interest reduction post-fix.
```

## Usage Rule
No debt item is "real" in planning until it has a TDR.
