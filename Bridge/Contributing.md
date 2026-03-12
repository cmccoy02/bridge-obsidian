# Contributing

[[00 - Home|← Home]] | See also: [[Bridge Architecture]], [[Roadmap]], [[Product Description]]

---

## Before you start

Read these three documents first. They'll save you from asking questions that are already answered, and they'll give you the context you need to make good decisions independently:

1. [[Problem Definition]] — what problem we're solving and why it matters now
2. [[Product Description]] — what the product is, how the interfaces fit together, and what `.bridge.json` is for
3. [[Bridge Architecture]] — the system design you must understand before touching any code

The architectural rule that matters most: **no intelligence in the interfaces.** The MCP server does not compute scores. The CLI does not evaluate gates. The desktop app does not detect languages. They all call `@bridge/core`. If you find yourself writing scoring or analysis logic anywhere other than `packages/core`, stop and move it.

---

## Repo structure

Bridge is a monorepo using pnpm workspaces:

```
bridge/
  packages/
    core/       ← @bridge/core — the brain. Pure Node.js, no UI, no transport
    mcp/        ← bridge-mcp — stdio MCP server, thin wrapper over core
    cli/        ← bridge-cli — terminal interface, thin wrapper over core
    desktop/    ← bridge-desktop — Electron GUI, thin wrapper over core
  apps/
    console/    ← bridge-console — Next.js web dashboard (Phase 2)
```

---

## Setup

Prerequisites: Node.js 20+, pnpm 8+.

```bash
# Clone the repo
git clone https://github.com/buildwithbridge/bridge.git
cd bridge

# Install all dependencies across all packages
pnpm install

# Build core first (everything else depends on it)
pnpm --filter @bridge/core build

# Build everything
pnpm build

# Run tests
pnpm test
```

To run the CLI locally during development:

```bash
cd packages/cli
pnpm dev
# or link it globally
pnpm link --global
bridge scan /path/to/some/repo
```

To run the MCP server locally and test it with Claude Code or Cursor, add it to your agent's MCP config:

```json
{
  "mcpServers": {
    "bridge": {
      "command": "node",
      "args": ["/path/to/bridge/packages/mcp/dist/index.js"],
      "env": {}
    }
  }
}
```

---

## How to work

### Where things live

| You want to... | Edit this |
|---|---|
| Change how debt is scored | `packages/core/src/techDebtScorer.ts` |
| Add a new detected pattern | `packages/core/src/securityPatterns.ts` or relevant dimension file |
| Modify gate evaluation logic | `packages/core/src/gateEvaluator.ts` |
| Add a new MCP tool | `packages/mcp/src/server.ts` — thin wrapper, call core |
| Add a new CLI command | `packages/cli/src/cli.ts` — thin wrapper, call core |
| Change the desktop dashboard | `packages/desktop/src/` |
| Change `.bridge.json` schema | `packages/core/src/bridgeConfig.ts` |

### Adding a new debt detection pattern

Every debt pattern Bridge detects lives in [[Bridge Knowledge Base]] and has a corresponding implementation in `@bridge/core`. When you add a new pattern, do both:

1. Add the pattern definition to `Bridge Knowledge Base.md`: severity, detection method, score impact, why it matters, fix strategy
2. Implement the detection in the relevant dimension file in `packages/core/src/`
3. Add tests in `packages/core/src/__tests__/`

The pattern ID format is `DIM-NNN` (e.g., `SEC-009`, `DEP-009`). Use the next available number in the relevant dimension.

### Writing tests

Test coverage is non-negotiable for `@bridge/core`. This is the brain of the product — untested scoring logic is a liability.

- Use vitest
- Unit tests live next to source files: `src/foo.ts` → `src/__tests__/foo.test.ts`
- Test with real-ish fixtures: create small sample repos in `packages/core/fixtures/` that represent specific debt states
- Never test implementation details; test behavior: "given this repo structure, the score should be X"

---

## Branching and PRs

- `main` is always deployable
- Feature branches: `feat/description`
- Bug fixes: `fix/description`
- Keep PRs small and focused. One concern per PR.
- PR description should answer: what does this change, why, and how do you verify it works?

Before opening a PR, run:

```bash
pnpm lint
pnpm test
pnpm build
```

All three must pass. CI will enforce this, but run it locally first to save time.

---

## Design decisions

Bridge runs on a few firm principles. Don't fight them; understand them first.

**`@bridge/core` is pure.** It has no Electron imports, no CLI framework imports, no HTTP server, no MCP SDK. It's a library. If you need to add a dependency to core, think hard about whether it's really necessary — every dependency in core is a dependency in every interface.

**Interfaces are thin.** The MCP server, CLI, and desktop app are all wrappers. Their job is to translate between the transport format and `@bridge/core` function calls. They should be boring.

**The `.bridge.json` format is the product.** This config file is what makes Bridge feel like a team member. It's machine-readable by agents, enforced by CI, and human-writable by developers. Changes to the schema need to be backwards-compatible — teams will have this file in their repos.

**Scoring changes need to be deliberate.** When you change a score weight or detection threshold, you're changing what every repo scores as. That has downstream effects on gates, on dashboard trends, on customer trust. Score changes should be discussed before implementation.

---

## Where to ask questions

If something is unclear, check these docs in order before asking:

1. [[Bridge Architecture]] — for system design questions
2. [[Bridge Knowledge Base]] — for questions about what we detect and why
3. [[Bridge MCP]] — for questions about the MCP server design
4. [[Roadmap]] — for questions about what we're building and what's deprioritized
5. Ask Connor directly if it's still not clear

---

## The culture

We're a small team building in a space that moves fast. A few things that matter:

- Be attached to the problem, not to your solution. If customer feedback says a feature isn't working, kill it and find something better. See [[Our Positioning]].
- Write code you'd be comfortable being scored by Bridge. Seriously.
- Keep the interfaces boring and the core sharp. Clever code in the wrong place is a liability.
- Document decisions. If you make a non-obvious architectural call, write a note in the relevant doc so the next person understands why.
