# Tech Debt - Debt Tracking System

Parent: [[tech debt]]

## Core Artifact: TDR (Technical Debt Record)
Each debt item must include:
- debt_id
- component/service
- debt_type (from taxonomy)
- owner
- origin_date
- principal_estimate
- interest_estimate
- risk_notes
- grade
- target_state
- repayment_plan
- review_date

## Workflow
1. Detect or declare debt.
2. Create/update TDR.
3. Grade debt.
4. Prioritize against roadmap.
5. Execute remediation and verify impact.
6. Close only when target-state metrics are met.

## Integrations
- Link TDRs to issue tracker epics.
- Link incidents to related debt items.
- Sync score changes to dashboard.

## Anti-Patterns
- Backlog-only debt tracking with no economics.
- Debt tickets without owners.
- Closing debt items without post-fix validation.
