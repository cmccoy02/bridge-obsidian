# Tech Debt - Taxonomy

Parent: [[tech debt]]

## Definition
Technical debt is the future cost and risk created by current implementation choices.

## Canonical Debt Types
- Code debt: low readability, high complexity, duplicated logic, weak boundaries.
- Architectural debt: structural decisions that constrain change and increase coordination cost.
- Test debt: missing or flaky tests, weak or slow verification.
- Documentation debt: stale/absent docs, missing rationale, tribal knowledge dependence.
- Data debt: schema drift, low quality data contracts, weak lineage and observability.
- Dependency debt: outdated libraries/platform versions, upgrade lock-in.
- Security debt: deferred vulnerability remediation, weak hardening, insecure defaults.
- Process debt: fragile release workflows, excessive manual steps, unclear ownership.

## AI-Era Extensions
- Prompt debt: undocumented prompts and system instructions that act like hidden code.
- Context debt: fragile retrieval/context packaging that breaks output reliability.
- Model debt: coupling to specific models/providers without abstraction or fallbacks.
- Evaluation debt: no durable eval harness for AI outputs.
- Verification debt: code generation speed exceeds review/testing capacity.

## Why This Matters
The key mistake is treating all debt as code cleanup. Most high-cost debt in scale systems is architectural, data, or coordination debt.

## Bridge Use
Use this taxonomy to tag every debt record with one primary and one secondary debt type.
