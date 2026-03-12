# Tech Debt - Management Operating Model

Parent: [[tech debt]]

## Roles
- Product: weighs debt against roadmap outcomes.
- Engineering manager: owns debt portfolio health.
- Tech lead/architect: sets debt standards and target states.
- Staff engineers: execute high-risk remediation.
- SRE/Sec: validate reliability and security outcomes.

## Cadence
- Weekly: component-level debt triage.
- Monthly: portfolio review and re-grading.
- Quarterly: debt-retirement planning linked to strategic goals.

## Governance Rules
- No critical debt without executive visibility.
- No high-grade debt without mitigation owner.
- Architecture debt requires decision records (ADR).
- AI feature debt requires eval evidence.

## KPI Set
- debt_interest_hours / sprint
- high-grade debt count trend
- incident recurrence by debt-linked component
- lead time and change failure rate for debt-heavy areas

## Operating Principle
Debt management is an operating system, not a side project.
