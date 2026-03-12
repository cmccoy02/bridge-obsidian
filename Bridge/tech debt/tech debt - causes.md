# Tech Debt - Causes

Parent: [[tech debt]]

## Primary Drivers
- Delivery pressure: short horizons force local optimization.
- Requirement volatility: teams implement quickly before domain stabilizes.
- Ownership gaps: unclear module ownership causes deferred cleanup.
- Skill mismatch: system complexity exceeds team architecture maturity.
- Incentive design: feature throughput rewarded, debt reduction invisible.

## Structural Causes
- Weak architecture decision records.
- Missing quality gates in CI/CD.
- Incomplete test pyramid and long feedback cycles.
- Lack of dependency lifecycle governance.
- Documentation not integrated into Definition of Done.

## AI-Specific Causes
- Over-trust in generated code with under-investment in verification.
- Faster PR volume than code-review capacity.
- Boilerplate proliferation from copy/paste generation.
- Inconsistent patterns caused by multi-agent prompt drift.
- Missing AI eval pipelines for reliability and security.

## Early Warning Indicators
- Rising hotfix ratio.
- Cycle time up while commit volume is stable.
- Increased merge conflicts in core modules.
- Repeated rollback incidents on the same services.
- Growing number of "temporary" TODO/FIXME items with no owner.

## Bridge Implication
Debt is usually caused by governance and flow design, not by individual developer quality.
