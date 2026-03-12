# Tech Debt - Architectural Debt

Parent: [[tech debt]]

## Definition
Architectural debt is debt embedded in system structure and design decisions that raises future change cost.

## Common Forms
- Layer violations and circular dependencies.
- Shared database schemas without bounded contexts.
- Over-centralized core services.
- API contracts with implicit, undocumented assumptions.

## Why It Is Expensive
Architecture debt multiplies coordination cost across teams and often drives recurring incident classes.

## Assessment Lenses
- Change coupling: how many modules change together?
- Dependency health: unstable or cyclic dependency graphs.
- Decision traceability: can teams explain why major structure exists?
- Blast radius: how many systems are affected by a single change?

## Evidence Signals
Recent studies connect architectural debt with measurable maintenance overhead and reduced agility, especially in resource-constrained teams.

## Repayment Strategies
- Strangler paths for high-risk legacy cores.
- Domain boundary enforcement (module/API policies).
- Incremental interface hardening with compatibility contracts.
- Architectural fitness functions in CI.
