
**The core insight is that Bridge has one brain and many interfaces.** The scoring engine, the config system, the gate evaluator, the dependency analyzer — that's the brain. The desktop app, the MCP server, the CLI, the GitHub App — those are all just different ways of talking to the same brain. If you architecture it that way, everything gets simpler.

Here's how I'd draw the map:

```
                    ┌──────────────────────────┐
                    │      @bridge/core        │
                    │                          │
                    │  bridgeConfig.ts         │
                    │  techDebtScorer.ts       │
                    │  gateEvaluator.ts        │
                    │  scanReport.ts           │
                    │  securityPatterns.ts     │
                    │  patchBatch.ts           │
                    │  analysis.ts             │
                    │                          │
                    │  Pure Node.js, no UI,    │
                    │  no transport, no runtime  │
                    │  assumptions.              │
                    └─────────┬────────────────┘
                              │
            ┌─────────────────┼─────────────────────┐
            │                 │                     │
     ┌──────┴──────┐  ┌──────┴──────┐  ┌──────────┴────────┐
     │ bridge-mcp  │  │ bridge-cli  │  │ bridge-desktop     │
     │             │  │             │  │ (Electron)         │
     │ stdio MCP   │  │ Terminal    │  │ GUI dashboard      │
     │ server      │  │ commands    │  │                    │
     └─────────────┘  └─────────────┘  └────────────────────┘
            │                 │
     Used by agents    Used by humans
     (Claude Code,     & CI pipelines
      Cursor, etc.)
```

**`@bridge/core`** is the real product. It's a plain Node.js library — no Electron dependencies, no CLI framework, no MCP SDK, no HTTP server. Just pure functions: give it a repo path, it gives you a score. Give it a config, it evaluates gates. This is what you'd publish as an npm package. Everything else imports from it.

**`bridge-mcp`** is a thin wrapper that exposes core through the MCP protocol. Separate repo or separate package within a monorepo. It depends on `@bridge/core` and `@modelcontextprotocol/sdk`. Nothing else.

**`bridge-cli`** is a thin wrapper that exposes core through terminal commands. `bridge scan`, `bridge score`, `bridge gates`, `bridge init`. It depends on `@bridge/core` and a CLI framework. This is what runs in CI and what developers use from their terminal.

**`bridge-desktop`** is the Electron GUI. It depends on `@bridge/core` and renders dashboards. This is the least important piece long-term — it's a nice visualization layer, but the value is in core, not in the GUI.

Now, the other services you listed:

**Bridge Console** is a web dashboard for engineering leaders. It consumes scan reports from multiple repos (uploaded by desktop, CLI, or CI). It's a separate web app (probably Next.js) with its own backend/database. It does NOT contain scoring logic — it receives pre-computed scores from core. Think of it as a read-only aggregation layer. Separate repo, separate deployment.

**GitHub App** listens for PRs and runs `@bridge/core` against them. It posts gate results as PR checks and comments with the debt score delta. It depends on `@bridge/core` plus GitHub's API. This is extremely high-value — it's how Bridge becomes part of every team's workflow without anyone changing their habits. Separate repo, deployed as a service.

**IDE Extension** (VS Code) — honestly, I'd deprioritize this. The MCP server already gives agents everything they need inside VS Code/Cursor. A dedicated extension would show the debt score in the status bar and highlight security findings inline, which is nice but not essential. The MCP server IS the IDE integration for the agent era. If you build this, it should just be a thin UI over `@bridge/core` running locally.

**CLI** — this is the second most important interface after MCP. It's how Bridge runs in CI (`bridge gates --fail-on-error`), how developers check things manually (`bridge score`), and how you demo the product. It should be the same npm package as core or a very thin wrapper.

**Private Registry** — I'd shelf this. It's a different business, a different technical challenge, and it dilutes focus. The `bridge_check_package` tool in MCP already gives you supply chain gating without building a registry. You can revisit this when you have 100 paying customers asking for it.

So the full picture, trimmed to what matters:

**Ship now:**

- `@bridge/core` (extracted from electron/services/)
- `bridge-mcp` (the agent interface — this is the growth engine)
- `bridge-cli` (the human/CI interface)

**Keep but deprioritize:**

- `bridge-desktop` (nice for demos, not the long-term value)

**Build next quarter:**

- Bridge Console (leadership dashboard)
- GitHub App (PR gates — this is the viral loop)

**Shelf:**

- Private registry (different problem)
- IDE extension (MCP replaces this)

The repo structure question: I'd go **monorepo**. One repo with packages:

```
bridge/
  packages/
    core/           ← @bridge/core
    mcp/            ← bridge-mcp
    cli/            ← bridge-cli
    desktop/        ← bridge-desktop (current electron app)
  apps/
    console/        ← bridge-console (Next.js dashboard)
```

This lets you share types, share the core library without publishing to npm on every change, and keep everything in sync. Use pnpm workspaces. When core is stable, you publish it to npm so the GitHub App (separate repo, separate deployment) can depend on it.

The key architectural rule: **no intelligence in the interfaces.** The MCP server doesn't compute scores. The CLI doesn't evaluate gates. The desktop app doesn't detect languages. They all call `@bridge/core`. If you find yourself writing scoring logic in the MCP server, you're doing it wrong — put it in core.

That's the map. The immediate move is extracting your `electron/services/` into a standalone `@bridge/core` that has zero Electron imports, then pointing both desktop and MCP at it.