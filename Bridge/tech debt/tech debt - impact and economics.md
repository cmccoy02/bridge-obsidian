# Tech Debt - Impact and Economics

Parent: [[tech debt]]

## Economic Model
- Principal: cost to remove debt now.
- Interest: recurring extra cost paid because debt remains.
- Risk premium: expected loss from incidents and failed changes.

## Practical Interest Formula
`interest_per_period = rework_hours + delay_hours + incident_hours + coordination_overhead`

## Observable Business Effects
- Slower feature throughput.
- Lower release reliability.
- Increased defect density and customer-facing incidents.
- Engineering attrition due to persistent friction.

## What Recent Evidence Suggests
- Architecture debt can consume meaningful portions of engineering effort in SMEs (JSS study).
- CASE 2024 reports links between architectural debt and significant maintenance overhead.
- AI-era studies suggest code creation speed increases, but quality/security assurance can lag.

## Portfolio Principle
Debt is rational when expected value from speed exceeds the discounted future interest. It becomes irrational when interest crosses the team’s throughput gain.

## Bridge Metric Minimum
Track at least:
- debt principal estimate
- debt interest estimate
- affected surface area
- time-to-remediate
- incident contribution
