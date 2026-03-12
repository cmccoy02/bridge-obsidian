# Tech Debt - Mitigation Playbook

Parent: [[tech debt]]

## Prevention
- Architecture fitness checks in CI.
- Definition of Done includes tests + docs + telemetry.
- Dependency update policy with time-boxed upgrades.
- Service ownership and escalation paths.

## Containment
- Isolate fragile components behind stable interfaces.
- Add feature flags and kill switches for risky flows.
- Increase observability for debt-heavy areas.

## Repayment Patterns
- Boy scout rule for touched files.
- Parallel run + strangler migration for legacy replacement.
- Test harness expansion before refactor.
- Data contract hardening before service decomposition.

## AI-Specific Mitigation
- Prompt and policy versioning.
- Mandatory eval gates for release.
- Risk tiering: stricter review for high-impact generated code.
- Security scanning tuned for generated code patterns.

## Success Criteria
Debt remediation is successful only if interest rate decreases measurably.
