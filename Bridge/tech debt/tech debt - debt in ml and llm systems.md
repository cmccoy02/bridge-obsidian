# Tech Debt - Debt in ML and LLM Systems

Parent: [[tech debt]]

## Foundational Insight
ML/LLM systems accumulate debt in pipelines, data, features, evaluation, and serving infrastructure, not only in model code.

## Recurring Debt Patterns
- Entanglement: behavior depends on hidden cross-feature interactions.
- Data dependencies: silent distribution drift breaks assumptions.
- Glue-code bloat: orchestration complexity grows faster than model logic.
- Configuration debt: unmanaged prompts/thresholds become hidden behavior.
- Eval debt: no stable benchmark for model revisions.

## LLM-Specific Risks
- Prompt regressions after small instruction edits.
- Retrieval fragility from schema or embedding changes.
- Model/provider lock-in with unclear fallback behavior.
- Safety-policy drift between environments.

## Practical Controls
- Version everything: prompts, context templates, eval sets, model configs.
- Keep golden-task suites with automatic diffing.
- Separate experimentation and production config paths.
- Attach observability to model behavior, not only infra metrics.

## Key References
- NeurIPS 2015 Hidden Technical Debt in ML Systems.
- Google Rules of ML.
- Recent studies on technical debt in AI-enabled systems.
