# Tech Debt - Bridge Product Opportunities

Parent: [[tech debt]]

## Product Wedge
Bridge can become the debt intelligence and execution platform for AI-era software teams.

## Opportunity 1: Debt Underwriting Engine
- Convert repo/CI/incident data into debt grades.
- Estimate principal/interest with confidence intervals.
- Recommend repayment sequence by ROI and risk.

## Opportunity 2: Verification Debt Monitor
- Track generated-code volume vs. review/test capacity.
- Trigger alerts when verification debt crosses thresholds.
- Auto-suggest mitigation plays (tests, targeted reviews, refactors).

## Opportunity 3: Architecture Debt Navigator
- Dependency + change-coupling graph with hotspot detection.
- Blast-radius forecasting before merges.
- Guided decomposition plans for high-interest domains.

## Opportunity 4: Documentation Debt Copilot
- Detect stale docs via code-doc divergence.
- Enforce runbook/ADR completion in PR workflows.
- Generate and validate operational documentation drafts.

## Opportunity 5: Debt Portfolio Dashboard for Leaders
- CFO/CTO-ready debt balance sheet view.
- Debt trend vs. velocity/reliability/business outcomes.
- Scenario modeling: "What if we repay top 10 debt items this quarter?"

## Differentiation
Most tools score code quality. Bridge should score debt economics + execution certainty.

## Immediate Experiments
1. Build a lightweight TDR ingestion + scoring workflow.
2. Pilot verification-debt alerts on one active repository.
3. Validate whether debt-interest estimates predict incident clusters.
