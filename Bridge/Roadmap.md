# Roadmap

[[HOME|← Home]] | See also: [[Product Description]], [[Bridge Architecture]], [[Contributing]]

---

## Current status

Bridge has a working core: the scoring engine, CLI, and Electron desktop app exist and can analyze real repos. The MCP server exists. The fundamental tech works.

The immediate priority is hardening the architecture so everything runs from a single shared brain (`@bridge/core`), then getting to a state where we can put it in front of paying customers and start learning.

See [[Bridge Architecture]] for the full technical design.

---

## Phase 1 — Foundation (current)

**Goal:** Stable, shippable core. Everything runs from `@bridge/core`. You can hand a repo URL to Bridge and get a reliable, meaningful score.

**Work:**

- Extract `@bridge/core` from Electron — pure Node.js library, zero UI dependencies, zero transport assumptions
- Point both the CLI and the desktop app at `@bridge/core` (stop duplicating logic)
- Ship `bridge-mcp` as a stable, stdio-transport MCP server that any Claude Code / Cursor user can configure in minutes
- Ship `bridge-cli` with core commands: `bridge scan`, `bridge score`, `bridge gates`, `bridge init`
- Establish `.bridge.json` config format as the standard for conventions and gates
- Monorepo structure: `packages/core`, `packages/mcp`, `packages/cli`, `packages/desktop`

**Definition of done:** A developer can `npm install -g bridge-cli`, run `bridge scan` on their repo, and get a debt score with actionable findings. An agent user can add Bridge MCP to their config, call `bridge_get_context`, and immediately have codebase conventions and debt score available in context.

---

## Phase 2 — First customers (next quarter)

**Goal:** 5–10 paying customers. Enough signal to know what matters.

**Work:**

- **GitHub App** — this is the viral loop. Listens for PRs, posts gate results as checks, comments with debt delta. Bridge enters team workflows without requiring anyone to install anything new. Target: every repo that installs the app runs Bridge on every PR.
- **Bridge Console** (v1) — web dashboard for engineering leaders. Shows score history across repos, trend lines, highest-priority findings. Separate Next.js app; consumes pre-computed scan reports from `@bridge/core`. This is what CTOs pay for.
- **Multi-language support** — current analysis is JS/TS-heavy. Expand to Python, Go, and Ruby to stop turning away potential customers.
- **Churn analysis** — cross-reference git commit history with debt findings. Files that change frequently and have low test coverage are the highest-risk items. This is the most predictive signal for future bugs.
- **Business impact framing** — show estimated developer hours wasted per finding. Turn a debt score into a dollar figure. This is what makes the product credible to executives.

**Definition of done:** A CTO can sign up, connect their GitHub org, and within 10 minutes see a debt score for every repo, trend lines, and a prioritized list of findings with estimated business impact. We have at least 5 companies paying us monthly.

---

## Phase 3 — Growth and depth (6+ months out)

**Goal:** Product-led growth. Bridge spreads through engineering orgs on its own.

**Work:**

- **VS Code extension** — thin UI over `@bridge/core`, shows debt score in status bar and highlights security findings inline. The MCP server already covers the agent use case; this is for developers who aren't using agents yet.
- **Automated remediation** — Bridge identifies the fix and opens a PR. Starting with the highest-confidence, lowest-risk changes: dependency updates, formatter enforcement, dead code removal, lockfile generation. Each fix is a separate PR with a clear explanation. Human reviews and merges.
- **Team benchmarking** — anonymized comparisons against similar companies (same stack, same size, same industry). "Your test coverage is in the bottom 20% of Series B fintech companies." This is the kind of context that gets executives to act.
- **Webhook and CI/CD integrations** — first-class support for CircleCI, Jenkins, GitLab CI, Bitbucket Pipelines. The GitHub App is step one; CI-agnostic is the long-term target.
- **API for enterprise** — some companies want to pull Bridge data into their own dashboards. A clean REST API makes Bridge data composable.

**Shelf (revisit with 100+ paying customers):**
- Private package registry — different business, different technical challenge, dilutes focus
- Native IDE extension beyond VS Code — MCP already covers the agent use case better

---

## Prioritization principles

**Agent integration over GUI.** The MCP server is the growth engine. It puts Bridge in front of every developer who uses an AI coding tool, without requiring them to open a new dashboard. Every hour spent on MCP compounds more than every hour spent on UI polish.

**Depth over breadth.** It's better to do JS/TS analysis extremely well than to do 10 languages mediocrely. Our score needs to be trustworthy before we expand the surface area.

**Business impact framing always.** Every finding needs to be expressible in developer hours or dollars. Technical output is not enough — the product needs to speak the language of resource allocation decisions.

**No scoring logic outside `@bridge/core`.** If you're writing analysis code anywhere other than `packages/core`, you're doing it wrong. Interfaces are thin; intelligence is centralized.

---

## What we're not building

- A code review tool (that's linear comments on PRs; we're portfolio-level health)
- A project management tool (we're not a Jira replacement)
- An AI code generator (we analyze code; we don't write it)
- A private package registry (completely different problem)

---

## Open questions

- **Pricing model**: per-seat vs. per-repo vs. per-organization? Current hypothesis: per-organization with repo limits by tier. Need to validate with customers.
- **Where does the GitHub App run?** Self-hosted vs. Bridge-hosted service. Self-hosted is simpler to ship; hosted is better for growth. Need to decide before Phase 2.
- **How do we handle monorepos?** Large organizations have monorepos with 50+ packages. Scoring at the repo level vs. package level is meaningfully different.
