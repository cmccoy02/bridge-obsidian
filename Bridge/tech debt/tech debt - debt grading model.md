# Tech Debt - Debt Grading Model

Parent: [[tech debt]]

## Purpose
Grade debt consistently so remediation decisions are defensible.

## Score Dimensions (1-5 each)
- Severity: technical fragility and defect likelihood.
- Interest Rate: recurring cost per sprint/month.
- Blast Radius: user/business impact surface.
- Reversibility: difficulty of safe rollback or fix.
- Confidence: evidence quality behind estimates.

## Composite Score
`debt_score = (severity * interest_rate * blast_radius * reversibility) / confidence`

## Grade Bands
- A (low): monitor only.
- B (moderate): bundle with adjacent feature work.
- C (material): schedule within quarter.
- D (high): priority remediation plan this cycle.
- E (critical): stop-the-line candidate.

## Stop-the-Line Triggers
- Repeated incidents with same root cause.
- Security-critical vulnerability with known exploit path.
- Debt item blocking compliance or contractual obligations.

## Governance
Re-grade monthly or after major incidents.
