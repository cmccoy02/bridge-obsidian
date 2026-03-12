# Tech Debt - Prioritization Framework

Parent: [[tech debt]]

## Priority Equation
`priority = (interest + risk_exposure + strategic_blocker_weight) / remediation_cost`

## Inputs
- Interest: recurring drag in time/cost.
- Risk exposure: incident/security/compliance impact.
- Strategic blocker: does debt block key roadmap bets?
- Remediation cost: engineering effort + migration risk.

## Portfolio Buckets
- Runway debt: blocks growth or reliability targets.
- Operational debt: creates support/on-call burden.
- Optionality debt: limits architecture evolution.
- Hygiene debt: low risk, opportunistic cleanup.

## Sequencing Policy
- Always fix top runway debt first.
- Pair medium debt fixes with touching-feature work.
- Reserve 10-30% capacity for planned debt retirement depending on incident pressure.

## Decision Boundary
If repayment ROI is unclear, run a short experiment: fix one high-interest slice and measure throughput/reliability change.
