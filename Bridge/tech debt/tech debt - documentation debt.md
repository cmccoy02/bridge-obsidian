# Tech Debt - Documentation Debt

Parent: [[tech debt]]

## Definition
Documentation debt is the gap between what people must know to safely change a system and what is actually captured.

## High-Impact Gaps
- Missing architecture decision records.
- Stale API and runbook docs.
- Missing operational playbooks.
- Untracked assumptions in prompts/configuration.

## Why This Debt Is Severe
Documentation debt creates hidden single points of failure: knowledge resides in people, not systems.

## Agile Reality
Research in agile contexts shows documentation debt is prevalent and often underestimated; organizations report significant impact on maintainability and knowledge transfer.

## Minimum Documentation Baseline
- ADR for all irreversible design decisions.
- Runbook for every tier-1 service.
- API contracts with examples and edge cases.
- On-call troubleshooting map (common failures + responses).
- AI prompt/eval changelog for productionized AI features.

## Bridge Rule
A change is incomplete if its operating knowledge is not captured.
