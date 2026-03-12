# Tech Debt - Manifestations

Parent: [[tech debt]]

## In Code
- High cyclomatic complexity, deep nesting, low cohesion.
- Duplicate logic across services.
- Ambiguous naming and weak boundaries.

## In Architecture
- God services and overloaded APIs.
- Tight coupling across domains.
- Cross-cutting changes requiring multi-team coordination.

## In Delivery Systems
- Slow CI/CD, brittle pipelines, manual releases.
- Frequent rollback and hotfix patterns.
- High lead time variance.

## In Documentation and Knowledge
- Runbooks missing for known incidents.
- Architecture docs stale relative to current implementation.
- Onboarding depends on specific senior engineers.

## In AI-Enabled Systems
- Prompt and policy logic outside version control.
- Evaluation results not reproducible across model versions.
- Frequent output-regression incidents after model upgrades.

## Diagnostic Heuristic
If an area is "always painful" and hard to change safely, that area is likely debt-bearing even if it still works.
