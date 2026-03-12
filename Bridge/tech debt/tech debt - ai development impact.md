# Tech Debt - AI Development Impact

Parent: [[tech debt]]

## Current Frontier Signal (2025-2026)
- AI assistants materially increase output velocity.
- Evidence on maintainability is mixed: some studies find parity with human-written code under controlled conditions, while industry reports show verification and security gaps in real workflows.

## What Is New vs. Classic Debt
- Debt can accumulate in prompts, context, and eval pipelines, not just source code.
- Generated code can amplify inconsistency if architectural patterns are not enforced.
- Teams can ship quickly but defer validation, creating a verification backlog.

## Verification Debt Pattern
`generation_rate > review_rate + test_capacity`
When this inequality holds for long periods, defect risk compounds.

## Guardrails That Work
- Require tests or explicit test-plan in AI-authored PRs.
- Enforce architecture linting and boundary checks.
- Add AI-specific eval suites (security, correctness, policy adherence).
- Use golden tasks and regression snapshots for model/prompt changes.
- Record model/version/prompt metadata for reproducibility.

## Decision Rule
Use AI for acceleration where requirements and interfaces are stable; reduce autonomy where domain risk is high and verification cost is extreme.
