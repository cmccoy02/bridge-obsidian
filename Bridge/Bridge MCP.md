

**The single insight that drives the whole design:** MCP has two speeds — fast reads and expensive actions. Most MCP servers get this wrong by making everything a tool (expensive). Bridge gets it right by making the things agents need _constantly_ into cached resources that return in milliseconds, and reserving tools for things that do real work.

**The five things that actually matter, in order:**

**`bridge_get_context`** is the product. Everything else is supporting cast. One tool call, the agent gets: debt score, critical issues, conventions, banned packages, preferred libraries, failing gates. 50ms, ~500 tokens. An agent calls this at the start of every task, and suddenly it's working _with_ the team's practices instead of against them. This single tool solves the "agents don't understand our codebase" problem.

**`bridge://conventions`** is the resource that means you never repeat yourself. Every time you've typed "use named exports" or "we use vitest not jest" or "don't use moment" into a prompt — that's a convention that should live in `.bridge.json` and get served to every agent automatically, forever. Write it once, enforce it everywhere.

**`bridge_check_gates`** creates the self-correction loop. Agent writes code → checks gates → sees it broke something → fixes it → checks again → passes → commits. No human intervention for mechanical quality issues. This is what turns agents from "code generators that need babysitting" into "autonomous contributors that respect repo policies."

**`bridge_check_package`** is the supply chain gate. This is quietly the most important safety feature. Agents love to `npm install` things. Without a gate, an agent working on your codebase might pull in moment.js, an unmaintained package, or something with known CVEs. With this tool, every new dependency gets policy-checked before it's added.

**`bridge_init`** is the onboarding. New repo, no config? One tool call generates a `.bridge.json` with auto-detected settings. This is how Bridge spreads — an agent working on a new repo calls `bridge_init`, and now that repo is Bridge-enabled.

**Implementation-wise,** the MCP server is a separate package (`bridge-mcp/`) that imports your existing core services. It's a thin layer — the actual intelligence lives in `techDebtScorer.ts`, `bridgeConfig.ts`, `gateEvaluator.ts`, etc. The MCP server just exposes those through the protocol. This means the Electron dashboard and the MCP server always produce identical results because they're calling the same functions.

**The key technical call:** stdio transport, not HTTP. Every local agent (Claude Code, Cursor, Cline) spawns MCP servers as child processes via stdio. That's what your users will use. HTTP transport is for when Bridge Console wants to query Bridge remotely — a later concern. Start with stdio, it's simpler and it's what the ecosystem expects.

One more thing — the spec includes the `annotations` on each tool (like `readOnlyHint: true`). These aren't cosmetic. They tell the agent client whether a tool has side effects. `bridge_get_context` with `readOnlyHint: true` means the agent can call it freely without asking the user for permission. `bridge_update_deps` without that hint means the agent will ask before running it. These annotations are how you make Bridge feel seamless rather than annoying.
