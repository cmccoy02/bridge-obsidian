[[tech debt]]
[[Index-Bridge]]

# Bridge Tech Debt Knowledge Base

## Version 0.1 — Foundation

## Maintained by: Bridge 

> This document is the structured knowledge core of Bridge. It defines what tech debt IS, how to categorize it, how to detect it, and how to fix it. Each pattern is machine-readable and informs Bridge's scoring algorithm, gate evaluation, and agent context.

---

# 1. Taxonomy of Technical Debt

Technical debt is any property of a codebase that increases the cost of future changes. It is not inherently bad — some debt is strategic (shipping fast with known shortcuts). It becomes a problem when it's invisible, accumulating, or blocking.

## 1.1 The Six Dimensions

Bridge measures debt across six dimensions. Each maps to a scoring dimension in the tech debt score (0-100, higher = more debt).

|Dimension|What It Measures|Weight (default)|
|---|---|---|
|Dependencies|Outdated packages, vulnerability exposure, supply chain risk|20%|
|Security|Known vulnerabilities, insecure code patterns, exposed secrets|25%|
|Architecture|Structural complexity, coupling, dead code, oversized modules|20%|
|Testing|Test coverage, test infrastructure, confidence in changes|20%|
|Documentation|README quality, API docs, changelogs, onboarding material|5%|
|Code Health|TODO backlog, linting, formatting, consistency, hygiene|10%|

---

# 2. Dependency Debt Patterns

## DEP-001: Outdated Patch Dependencies

- **Severity**: Low
- **Detection**: `npm outdated --json` where semver diff is patch
- **Score Impact**: +1 per package
- **Why It Matters**: Patch updates are bug fixes and security patches. Falling behind means running code with known bugs. Low individual risk but compounds across many packages.
- **Fix Strategy**: Auto-update via `npm update` or Bridge PatchBatch. Patch updates are semver-safe.
- **Automatable**: Yes
- **Fix Command**: `npm update`

## DEP-002: Outdated Minor Dependencies

- **Severity**: Medium
- **Detection**: `npm outdated --json` where semver diff is minor
- **Score Impact**: +2 per package
- **Why It Matters**: Minor updates add features and may include performance improvements. Extended drift means missing optimizations and falling out of step with ecosystem documentation.
- **Fix Strategy**: Review changelogs, update in batches, run tests. Bridge PatchBatch automates this.
- **Automatable**: Yes, with test validation
- **Fix Command**: `npm install <package>@latest` per package

## DEP-003: Outdated Major Dependencies

- **Severity**: High
- **Detection**: `npm outdated --json` where semver diff is major
- **Score Impact**: +5 per package
- **Why It Matters**: Major version drift is the most expensive form of dependency debt. The longer you wait, the harder the migration. Multiple major versions behind can require rewriting integration code. Also correlates strongly with vulnerability exposure.
- **Fix Strategy**: One at a time, isolated branches, thorough testing. Read migration guides. Never batch major updates.
- **Automatable**: Partially — Bridge can identify and prepare, human reviews migration.

## DEP-004: Dependencies with Known Vulnerabilities

- **Severity**: Critical (if critical CVE), High (if high CVE)
- **Detection**: `npm audit --json`
- **Score Impact**: +20 per critical, +10 per high, +3 per medium, +1 per low
- **Why It Matters**: Known vulnerabilities are the highest-risk form of debt because they have public exploits. Unlike other debt which slows you down, this debt can get you breached.
- **Fix Strategy**: `npm audit fix` for auto-fixable, manual update for breaking changes. Prioritize by CVSS score.
- **Automatable**: Yes for non-breaking fixes
- **Fix Command**: `npm audit fix`

## DEP-005: Unused Dependencies

- **Severity**: Low-Medium
- **Detection**: `depcheck` or `knip` — packages in package.json not imported anywhere
- **Score Impact**: +1 per unused package
- **Why It Matters**: Every unused dependency increases install time, attack surface, and cognitive load. An unused package with a vulnerability still shows up in audits and still needs updating.
- **Fix Strategy**: Remove from package.json. Verify nothing breaks (some packages are used implicitly via plugins or config).
- **Automatable**: Partially — detection is automated, removal needs verification

## DEP-006: Duplicate Dependencies

- **Severity**: Medium
- **Detection**: `npm ls --all` showing multiple versions of the same package
- **Score Impact**: +2 per duplicate
- **Why It Matters**: Duplicates increase bundle size and can cause subtle bugs when different parts of the app use different versions of the same library. Common with lodash, React, and TypeScript types.
- **Fix Strategy**: Use `npm dedupe`, align version ranges in package.json, or use overrides/resolutions.

## DEP-007: Deprecated Dependencies

- **Severity**: High
- **Detection**: `npm view <package> --json` checking deprecated field
- **Score Impact**: +8 per deprecated package
- **Why It Matters**: A deprecated package will never receive security patches. It's a ticking time bomb. The maintainer has explicitly said "stop using this."
- **Fix Strategy**: Migrate to the recommended replacement. If no replacement, evaluate alternatives.

## DEP-008: Missing Lockfile

- **Severity**: High
- **Detection**: Absence of package-lock.json, yarn.lock, or pnpm-lock.yaml
- **Score Impact**: +15
- **Why It Matters**: Without a lockfile, every install can produce different dependency trees. This causes "works on my machine" bugs and makes builds non-reproducible. In CI, this means you might deploy different code than you tested.
- **Fix Strategy**: Run `npm install` and commit the lockfile.
- **Automatable**: Yes
- **Fix Command**: `npm install` (generates lockfile)

---

# 3. Security Debt Patterns

## SEC-001: Dynamic Code Execution (eval/Function)

- **Severity**: Critical
- **CWE**: CWE-94 (Code Injection)
- **OWASP**: A03:2021 (Injection)
- **Detection**: Regex — `\b(eval\s*\(|new\s+Function\s*\()`
- **Score Impact**: +8 per occurrence (capped at 40)
- **Why It Matters**: `eval()` and `new Function()` execute arbitrary strings as code. If any user input reaches these functions, the application is fully compromised.
- **Fix Strategy**: Replace with safe alternatives — JSON.parse for data, explicit maps for dynamic dispatch, safe expression parsers for math.
- **Languages**: JavaScript, TypeScript

## SEC-002: Command Injection via child_process

- **Severity**: Critical
- **CWE**: CWE-78 (OS Command Injection)
- **OWASP**: A03:2021 (Injection)
- **Detection**: Regex — `child_process\.exec\s*\(` with string concatenation/interpolation
- **Score Impact**: +8 per occurrence (capped at 40)
- **Why It Matters**: `exec()` runs commands through a shell. If user input is concatenated into the command string, attackers can inject arbitrary shell commands (e.g., `; rm -rf /`).
- **Fix Strategy**: Use `execFile()` or `spawn()` with explicit argument arrays. Never pass user input through a shell.
- **Languages**: JavaScript, TypeScript (Node.js)

## SEC-003: Hardcoded Secrets

- **Severity**: Critical
- **CWE**: CWE-798 (Use of Hard-coded Credentials)
- **OWASP**: A07:2021 (Identification and Authentication Failures)
- **Detection**: Regex patterns for AWS keys (AKIA...), GitHub tokens (ghp_...), Stripe keys (sk_live_...), PEM private keys, and generic `api_key = "..."` patterns
- **Score Impact**: +15 per occurrence (capped at 40)
- **Why It Matters**: Credentials in source code get committed to git history, shared across environments, and leaked in logs. They cannot be rotated without changing code.
- **Fix Strategy**: Move to environment variables, secret managers (AWS SSM, Vault, Doppler), or .env files excluded from git. Rotate any exposed credentials immediately.
- **Languages**: All

## SEC-004: XSS Sinks (innerHTML/dangerouslySetInnerHTML)

- **Severity**: High
- **CWE**: CWE-79 (Cross-site Scripting)
- **OWASP**: A03:2021 (Injection)
- **Detection**: Regex — `innerHTML\s*=|dangerouslySetInnerHTML`
- **Score Impact**: +5 per occurrence (capped at 40)
- **Why It Matters**: Assigning untrusted content to innerHTML allows attackers to inject scripts that steal cookies, redirect users, or modify the page.
- **Fix Strategy**: Use `textContent` for plain text. Use DOMPurify or a framework's built-in sanitization for HTML content. In React, avoid `dangerouslySetInnerHTML` unless content comes from a trusted, sanitized source.
- **Languages**: JavaScript, TypeScript

## SEC-005: SQL Injection via String Concatenation

- **Severity**: High
- **CWE**: CWE-89 (SQL Injection)
- **OWASP**: A03:2021 (Injection)
- **Detection**: Regex — SQL keywords followed by string concatenation/interpolation
- **Score Impact**: +5 per occurrence (capped at 40)
- **Why It Matters**: Concatenating user input into SQL queries allows attackers to read, modify, or delete database contents. One of the oldest and most exploited vulnerability classes.
- **Fix Strategy**: Use parameterized queries / prepared statements. ORMs like Prisma, Drizzle, or Sequelize handle this automatically.
- **Languages**: JavaScript, TypeScript, Python

## SEC-006: Weak Cryptographic Functions

- **Severity**: Medium
- **CWE**: CWE-327 (Use of Broken Crypto Algorithm)
- **OWASP**: A02:2021 (Cryptographic Failures)
- **Detection**: Regex — `md5\(|sha1\(|createHash.*['\"](md5|sha1)`
- **Score Impact**: +3 per occurrence
- **Why It Matters**: MD5 and SHA1 have known collision attacks. They should not be used for any security purpose (password hashing, integrity verification, signatures).
- **Fix Strategy**: Use SHA-256/512 for hashing. Use bcrypt, argon2, or scrypt for password hashing. Use HMAC for message authentication.
- **Languages**: All

## SEC-007: Insecure Randomness

- **Severity**: Medium
- **CWE**: CWE-330 (Insufficiently Random Values)
- **OWASP**: A02:2021 (Cryptographic Failures)
- **Detection**: Regex — `Math\.random\s*\(`
- **Score Impact**: +3 per occurrence
- **Why It Matters**: Math.random() is not cryptographically secure. Using it for tokens, session IDs, or any security-sensitive value makes them predictable.
- **Fix Strategy**: Use `crypto.randomBytes()`, `crypto.randomUUID()`, or Web Crypto `getRandomValues()`.
- **Languages**: JavaScript, TypeScript

## SEC-008: Disabled SSL/TLS Verification

- **Severity**: High
- **CWE**: CWE-295 (Improper Certificate Validation)
- **OWASP**: A02:2021 (Cryptographic Failures)
- **Detection**: Regex — `rejectUnauthorized\s*:\s*false|NODE_TLS_REJECT_UNAUTHORIZED\s*=\s*['"]?0`
- **Score Impact**: +8 per occurrence
- **Why It Matters**: Disabling certificate verification opens the application to man-in-the-middle attacks. All HTTPS traffic becomes interceptable.
- **Fix Strategy**: Fix the underlying certificate issue (expired cert, self-signed in dev) rather than disabling verification. Use proper CA bundles.
- **Languages**: JavaScript, TypeScript (Node.js)

---

# 4. Architecture Debt Patterns

## ARC-001: Circular Dependencies

- **Severity**: High
- **Detection**: `madge --circular` or `dependency-cruiser`
- **Score Impact**: +8 per cycle
- **Why It Matters**: Circular dependencies make code hard to reason about, hard to test in isolation, and can cause initialization order bugs. They indicate unclear module boundaries — the modules don't have a clean hierarchy.
- **Fix Strategy**: Identify the shared concept that both modules depend on and extract it into a third module. Use dependency injection to break the cycle. Refactor to follow a clear layering (e.g., services → models → utils, never backwards).
- **Common Causes**: Two modules importing from each other, barrel files (index.ts) re-exporting everything creating implicit cycles, type imports that create runtime dependencies.

## ARC-002: God Files (>500 lines)

- **Severity**: Medium
- **Detection**: File line count > 500
- **Score Impact**: +3 per file over 500 lines, +5 additional per file over 1000 lines
- **Why It Matters**: Large files are hard to navigate, hard to review, and hard to test. They tend to accumulate responsibilities over time (violating single responsibility). Multiple developers working in the same large file causes merge conflicts.
- **Fix Strategy**: Identify distinct responsibilities within the file. Extract related functions/classes into focused modules. Common extractions: separate data fetching from rendering, separate validation from business logic, separate types/interfaces into their own files.
- **Thresholds**: 300 lines for React components, 500 lines for services/utilities, 800 lines for complex business logic (with justification).

## ARC-003: Deep Nesting (>4 levels)

- **Severity**: Medium
- **Detection**: Indentation analysis or AST parsing
- **Score Impact**: +3 per file with nesting > 4 levels
- **Why It Matters**: Deeply nested code is hard to read and indicates that a function is doing too many things. It's a proxy for cyclomatic complexity.
- **Fix Strategy**: Use early returns/guard clauses to reduce nesting. Extract inner logic into helper functions. Replace nested conditionals with lookup tables or strategy patterns.
- **Example Anti-pattern**: if → if → for → if → try (5 levels)
- **Example Fix**: Guard clause at top, extracted loop body, flat structure

## ARC-004: Dead Code / Unused Exports

- **Severity**: Low-Medium
- **Detection**: `knip` or `unimported` for JS/TS
- **Score Impact**: +2 per dead file, +1 per unused export
- **Why It Matters**: Dead code is cognitive overhead — developers read it, try to understand it, and maintain it, but it serves no purpose. It also makes refactoring harder because you can't tell if something is "unused" or "used somewhere you haven't found yet."
- **Fix Strategy**: Delete it. If you're nervous, check git blame — if it hasn't been modified in 6+ months and nothing imports it, it's safe to remove. Git history preserves the code if you ever need it back.

## ARC-005: Tight Coupling (High Fan-In/Fan-Out)

- **Severity**: High
- **Detection**: Dependency graph analysis — files imported by >20 other files (high fan-in) or files importing from >15 other files (high fan-out)
- **Score Impact**: +5 per file exceeding thresholds
- **Why It Matters**: High fan-in means a change to this file affects many consumers — it's a risk amplifier. High fan-out means this file depends on many things — it's fragile and breaks when any dependency changes.
- **Fix Strategy**: For high fan-in: stabilize the API, add comprehensive tests, consider if the module should be split. For high fan-out: extract focused interfaces, use dependency injection, restructure to reduce surface area.

## ARC-006: Inconsistent Project Structure

- **Severity**: Low
- **Detection**: Heuristic — no clear src/ directory, mixed file types in root, no separation between components/services/utils
- **Score Impact**: +5
- **Why It Matters**: Inconsistent structure makes it hard for new developers (and agents) to find things. It signals that nobody is maintaining the architectural vision.
- **Fix Strategy**: Establish and document a directory convention. Common patterns: src/components, src/services, src/utils, src/types for frontend; src/routes, src/controllers, src/models, src/middleware for backend.

---

# 5. Testing Debt Patterns

## TST-001: No Test Infrastructure

- **Severity**: Critical
- **Detection**: No test command in package.json scripts, no test framework in dependencies
- **Score Impact**: +40
- **Why It Matters**: Without test infrastructure, there's no way to verify that changes don't break existing behavior. Every change is a gamble. This is the single most impactful form of technical debt because it makes ALL other debt harder to fix safely.
- **Fix Strategy**: Add a test framework (vitest for modern projects, jest for established ones). Write tests for the most critical paths first — don't try to get 100% coverage immediately.
- **Automatable**: Yes
- **Fix Command**: `npm install -D vitest` then add `"test": "vitest"` to package.json scripts

## TST-002: No Test Files

- **Severity**: High
- **Detection**: No files matching _.test._, _.spec._, or **tests**/ directory
- **Score Impact**: +30
- **Why It Matters**: Test infrastructure exists but nobody is writing tests. This is worse than TST-001 in some ways because the team has the tools but isn't using them.
- **Fix Strategy**: Start with integration tests for the most business-critical flows. One test that covers the happy path is better than zero tests.

## TST-003: Low Coverage on High-Churn Files

- **Severity**: High
- **Detection**: Cross-reference git log frequency with coverage report
- **Score Impact**: Variable — proportional to churn * (1 - coverage)
- **Why It Matters**: Files that change frequently are the most likely to have regressions. If they also have low test coverage, every change is high risk. This is the highest-signal debt metric for predicting future bugs.
- **Fix Strategy**: Prioritize tests for the 10 most-frequently-changed files. Focus on behavior tests (what does this function do?) not implementation tests (how does it do it?).

## TST-004: Flaky Tests

- **Severity**: High
- **Detection**: CI logs showing tests that pass/fail inconsistently
- **Score Impact**: +10 per known flaky test
- **Why It Matters**: Flaky tests erode trust in the test suite. When developers see random failures, they start ignoring test results entirely. A flaky test suite is almost worse than no tests because it teaches the team that failures don't mean anything.
- **Fix Strategy**: Quarantine flaky tests, fix the root cause (usually timing, shared state, or external dependencies), then un-quarantine. Common fixes: mock external services, use deterministic time, isolate test state.

---

# 6. Documentation Debt Patterns

## DOC-001: Missing README

- **Severity**: High
- **Detection**: No README.md at repo root
- **Score Impact**: +30
- **Why It Matters**: The README is the entry point for every new developer and every agent. Without it, onboarding is verbal/tribal knowledge that doesn't scale.
- **Fix Strategy**: Write a README covering: what the project does (1 paragraph), how to install/run, how to test, how to contribute, architecture overview.

## DOC-002: Outdated README

- **Severity**: Medium
- **Detection**: README.md last modified date > 90 days, or content doesn't match current project state
- **Score Impact**: +10
- **Why It Matters**: An outdated README is worse than no README in some ways — it actively misleads. Agents and developers follow wrong instructions.
- **Fix Strategy**: Review and update quarterly. Add a "Last Updated" date to the README.

## DOC-003: Missing API Documentation

- **Severity**: Medium
- **Detection**: Exported functions without JSDoc/TSDoc comments
- **Score Impact**: +1 per undocumented exported function (capped at 15)
- **Why It Matters**: Undocumented APIs force consumers to read implementation code to understand behavior. This is expensive and error-prone.
- **Fix Strategy**: Document public APIs with JSDoc. Focus on: what the function does, what parameters mean, what it returns, what can go wrong.

---

# 7. Code Health Debt Patterns

## HLT-001: TODO/FIXME Backlog

- **Severity**: Low (under 10), Medium (10-30), High (over 30)
- **Detection**: Grep for TODO, FIXME, HACK, XXX comments
- **Score Impact**: +10 if count > 10, +10 additional if count > 30
- **Why It Matters**: TODO comments are promises to your future self. A small number is normal. A large number indicates that the team is consistently deferring work rather than completing it. Over 30 TODOs means the codebase has become a graveyard of good intentions.
- **Fix Strategy**: Triage the list. Delete stale TODOs (over 6 months). Convert important ones to issues. Fix the quick ones (< 15 minutes each).

## HLT-002: No Linter Configured

- **Severity**: Medium
- **Detection**: No eslint/biome/oxlint config file, no lint script in package.json
- **Score Impact**: +10
- **Why It Matters**: Without a linter, code style varies by developer, inconsistencies accumulate, and common mistakes go uncaught. Agents writing code without linting context produce inconsistent output.
- **Fix Strategy**: Add ESLint with a standard config (typescript-eslint for TS projects). Modern alternative: Biome (faster, fewer config headaches).
- **Automatable**: Yes
- **Fix Command**: `npm init @eslint/config` or `npx @biomejs/biome init`

## HLT-003: No Formatter Configured

- **Severity**: Low
- **Detection**: No prettier/biome format config, no format script
- **Score Impact**: +5
- **Why It Matters**: Without formatting, PRs contain noise — whitespace changes, quote style changes, trailing comma disagreements. This makes code review harder and git history noisier.
- **Fix Strategy**: Add Prettier or Biome. Configure once. Add a pre-commit hook.

## HLT-004: Console.log Statements in Production Code

- **Severity**: Low
- **Detection**: Grep for console.log (excluding test files and config)
- **Score Impact**: +1 per 10 occurrences (capped at 10)
- **Why It Matters**: console.log statements leak information, slow down the UI (especially in loops), and indicate code that was debugged manually and not cleaned up.
- **Fix Strategy**: Use a proper logger (pino, winston) for server code. Remove console.log from client code or use a debug flag. Add a lint rule to catch it.

## HLT-005: Missing .gitignore

- **Severity**: Medium
- **Detection**: No .gitignore file at repo root
- **Score Impact**: +5
- **Why It Matters**: Without .gitignore, node_modules, build artifacts, .env files, and editor config get committed. This bloats the repo, leaks secrets, and causes conflicts.
- **Fix Strategy**: Add a .gitignore using a standard template for the project's language/framework.

---

# 8. Cross-Cutting Patterns

## XCT-001: No CI/CD Configuration

- **Severity**: High
- **Detection**: No .github/workflows, .gitlab-ci.yml, Jenkinsfile, etc.
- **Score Impact**: +10
- **Why It Matters**: Without CI, tests don't run automatically, linting doesn't enforce, and deployments are manual. This means quality gates exist but aren't enforced.
- **Fix Strategy**: Add a basic CI config that runs tests and lint on every PR. GitHub Actions is the easiest starting point for most projects.

## XCT-002: Agent-Generated Code Without Review Gates

- **Severity**: High (emerging pattern — 2026)
- **Detection**: High volume of commits with AI-style patterns (large uniform PRs, perfect formatting, no incremental debugging commits), combined with no CI or code review requirements
- **Score Impact**: +15
- **Why It Matters**: AI-generated code is "highly functional but systematically lacking in architectural judgment" (Ox Security, 2025). Without review gates, structural problems accumulate invisibly. The code works today but becomes unmaintainable within months.
- **Fix Strategy**: Require PR reviews for all changes. Add Bridge gate checks to CI. Ensure agents have access to .bridge.json conventions before generating code.

---

# 9. Scoring Calibration Notes

## Benchmark Expectations (TypeScript/React, 10k-100k lines)

These are initial calibrations based on expert assessment. They will be refined with empirical data as Bridge scans more repos.

|Score Range|Grade|What It Looks Like|
|---|---|---|
|0-15|A|Well-maintained, all deps current, good test coverage, clean architecture, active CI|
|16-30|B|Some outdated deps, minor structural issues, tests exist but incomplete, generally healthy|
|31-50|C|Notable dependency drift, some security issues, architectural concerns, testing gaps|
|51-70|D|Significant debt across multiple dimensions, migrations overdue, testing insufficient|
|71-100|F|Critical security issues, major version drift, no tests, structural problems blocking development|

## Known Scoring Limitations (v0.1)

- No coverage percentage (only detects presence/absence of tests)
- No git-history-based churn analysis
- No bundle size tracking
- No complexity metrics (cyclomatic, cognitive)
- Architecture scoring based on file size only (no coupling/cohesion analysis)
- No duplicate code detection
- Trend requires historical data not yet collected

---

# 10. References and Sources

- Vercel React Best Practices (2026): 40+ performance rules, impact-ordered
- Ox Security "Army of Juniors" Report (2025): 10 anti-patterns in AI-generated code
- GitClear Analysis (2025-2026): 153M lines analyzed, documenting quality degradation
- Carnegie Mellon Study (2025): 800+ GitHub repos, AI tool impact on debt metrics
- Stack Overflow Developer Survey (2025): Trust vs. adoption paradox
- OWASP Top 10 (2021): Security vulnerability classification
- CWE Database: Common Weakness Enumeration for pattern identification
- SonarQube Rules: Reference for code quality rules (adapted for Bridge's taxonomy)