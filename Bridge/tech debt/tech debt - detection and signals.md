# Tech Debt - Detection and Signals

Parent: [[tech debt]]

## Detection Stack
- Static analysis: complexity, duplication, coverage, vulnerability trends.
- Repository mining: hotspots (high churn + low quality), change coupling.
- Operational telemetry: incident clusters, rollback frequency, MTTR by component.
- Socio-technical signals: review latency, ownership diffusion, onboarding time.

## SATD (Self-Admitted Technical Debt)
Mine TODO/FIXME/debt annotations but require owner + due date or they become debt noise.

## AI-Specific Detection
- Missing eval coverage for AI behaviors.
- Regression in golden tasks after model or prompt updates.
- Unreviewed high-risk generated code paths.
- Prompt/config drift without version tags.

## Practical Pipeline
1. Score each component weekly.
2. Detect hotspots where debt and churn overlap.
3. Open or update TDR automatically.
4. Trigger governance rules when threshold breaches occur.

## Suggested Tooling Inputs
- SonarQube metrics.
- Git history and PR metadata.
- Incident platform exports.
- Test/eval coverage reports.
