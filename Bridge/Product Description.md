# Product Description

[[00 - Home|← Home]] | See also: [[Problem Definition]], [[Bridge Architecture]], [[Bridge Knowledge Base]]

---

## What Bridge is

Bridge is a technical debt intelligence platform. It scans a codebase, produces a scored health report across six dimensions, and integrates into every workflow an engineering team already uses — the terminal, the AI coding agent, GitHub PRs, and an executive-facing dashboard.

The core insight: **Bridge has one brain and many interfaces.** The scoring engine lives in `@bridge/core`. The desktop app, the MCP server, the CLI, and the GitHub App are all just different ways to talk to that same brain. Every interface produces identical results because they call the same functions.

---

## Who it's for

Bridge serves two audiences simultaneously, and the product only works if it serves both:

**Engineering teams** (developers, tech leads, CTOs) use Bridge to get an honest, data-driven picture of codebase health. They need something that doesn't require manual input, integrates into existing tools, and gives them specific, actionable items — not vague warnings.

**Non-technical stakeholders** (PMs, executives, finance) use Bridge to understand risk and make resource allocation decisions. They need a score they can look at without reading code, a trend line they can track quarter over quarter, and plain-language framing around business impact (delayed releases, dev hours wasted, security exposure).

The product design lives at the intersection: rigorous enough that engineers trust it, clear enough that executives act on it.

---

## The six dimensions

Bridge scores debt from 0–100 (higher = more debt) across six dimensions. Each has a default weight; teams can adjust weights in `.bridge.json` to match their priorities.

| Dimension | What it measures | Default weight |
|---|---|---|
| Security | Known vulnerabilities, insecure patterns, exposed secrets | 25% |
| Dependencies | Outdated packages, vulnerability exposure, supply chain risk | 20% |
| Architecture | Structural complexity, coupling, dead code, oversized modules | 20% |
| Testing | Test coverage, test infrastructure, confidence in changes | 20% |
| Code Health | TODO backlog, linting, formatting, consistency, hygiene | 10% |
| Documentation | README quality, API docs, changelogs, onboarding material | 5% |

Full pattern definitions for every detected issue live in [[Bridge Knowledge Base]].

---

## The interfaces

### `@bridge/core` — the brain
A pure Node.js library. No UI, no transport, no runtime assumptions. Give it a repo path, it gives you a score. Give it a config, it evaluates gates. This is the real product. Everything else imports from it.

### Bridge CLI — for developers and CI
`bridge scan`, `bridge score`, `bridge gates`, `bridge init`. Runs in the terminal and in CI pipelines. When a developer or a pipeline wants to know the debt score of a repo, this is the tool. Gate enforcement (`bridge gates --fail-on-error`) is how Bridge blocks broken code from merging.

### Bridge MCP — for AI agents
The most important growth surface. Exposes `@bridge/core` through the Model Context Protocol, which means every AI coding agent (Claude Code, Cursor, Cline) can call Bridge tools directly from inside their workflow.

The five MCP tools, in order of importance:

1. **`bridge_get_context`** — One call returns debt score, critical issues, conventions, banned packages, and failing gates. 50ms, ~500 tokens. An agent calls this at the start of every task and immediately understands the repo's practices and health. This is the tool that solves "agents don't understand our codebase."

2. **`bridge://conventions`** (resource) — Serves the team's coding conventions from `.bridge.json` automatically to every agent, forever. Write conventions once, enforce them everywhere.

3. **`bridge_check_gates`** — Creates a self-correction loop. Agent writes code → checks gates → sees it broke something → fixes it → checks again → passes → commits. No human needed for mechanical quality issues.

4. **`bridge_check_package`** — Supply chain gate. Agents love to `npm install` things. This tool policy-checks every new dependency before it's added. Quietly the most important safety feature.

5. **`bridge_init`** — Onboarding. New repo, no config? One call generates a `.bridge.json` with auto-detected settings. This is how Bridge spreads virally.

See [[Bridge MCP]] for full design notes.

### Bridge Desktop — for demos and leaders
An Electron GUI that renders the debt score, trend history, and actionable insights as a dashboard. This is the most visible part of Bridge but not the most important. Its primary job right now is demos and executive buy-in. Long term it becomes the self-serve dashboard for engineering leaders.

### Bridge Console — leadership view (next quarter)
A web dashboard for engineering leaders with multiple repos. Consumes pre-computed scan reports; no scoring logic lives here. This is the product that CTOs and VPs of Engineering pay for — portfolio-level visibility into codebase health across teams.

### GitHub App — the viral loop (next quarter)
Listens for pull requests, runs `@bridge/core` against them, posts gate results as PR checks, and comments with the debt score delta. This is extremely high-value: Bridge becomes part of every team's workflow without anyone changing their habits. The GitHub App is how Bridge gets to developers who never heard of Bridge.

---

## The `.bridge.json` config

Every Bridge-enabled repo has a `.bridge.json` at the root. It defines:

- **Gates**: debt score thresholds that can block CI (`"maxScore": 70`)
- **Conventions**: team coding standards served to agents automatically
- **Banned packages**: packages that should never be installed
- **Preferred libraries**: the team's approved choices (e.g., "use vitest not jest")
- **Dimension weights**: custom scoring priorities

This file is the connective tissue between the scoring engine and the team's actual practices. It's what makes Bridge feel like a team member rather than a generic tool.

---

## Scoring calibration

| Score | Grade | What it looks like |
|---|---|---|
| 0–15 | A | Well-maintained, all deps current, good test coverage, active CI |
| 16–30 | B | Some outdated deps, minor structural issues, tests exist but incomplete |
| 31–50 | C | Notable dependency drift, some security issues, testing gaps |
| 51–70 | D | Significant debt across multiple dimensions, migrations overdue |
| 71–100 | F | Critical security issues, major version drift, no tests, blocking development |

---

## What Bridge is not

Bridge does not replace engineers. It doesn't write code, it doesn't merge PRs automatically, and it doesn't make architectural decisions. It makes the invisible visible — so humans (and agents) can make better decisions with better information.

Bridge also isn't a CI replacement. It's a layer on top of CI that adds intelligence: not just "tests passed" but "did this change increase or decrease codebase health, and by how much?"
