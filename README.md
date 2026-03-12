# bridge-obsidian

Internal knowledge base for Bridge — product docs, architecture decisions, strategy, and onboarding resources. Built as an Obsidian vault so everything is interconnected and explorable.

---

## Getting Started

### 1. Install Obsidian

Download and install Obsidian:
👉 [https://obsidian.md/download](https://obsidian.md/download)

---

### 2. Clone the Repo

```bash
git clone https://github.com/[bridge-org]/bridge-obsidian.git
cd bridge-obsidian
```

> Make sure you're using the correct GitHub account. If you're managing multiple accounts, confirm your git config is pointing to the right one:
> ```bash
> git config user.email
> ```

---

### 3. Open as a Vault in Obsidian

1. Open Obsidian
2. Click **"Open folder as vault"**
3. Navigate to and select the cloned `bridge-obsidian` folder
4. Click **Open**

---

### 4. Trust and Enable Plugins

When the vault opens, Obsidian will ask if you trust the vault author:

- Click **"Trust author and enable plugins"**

This enables the community plugins already configured for this vault (things like graph enhancements, dataview, etc.). Without this step, some views and features won't work correctly.

---

### 5. Explore the Vault

**Graph View** is the best way to get oriented. Open it with:
- `Cmd + Shift + G` (Mac) / `Ctrl + Shift + G` (Windows)

Or click the graph icon in the left sidebar. You'll see how notes connect across product, engineering, and strategy.

From there, explore:
- `00 - Start Here/` — orientation and key docs
- `Product/` — vision, roadmap, feature specs
- `Engineering/` — architecture decisions, MCP server, `.bridge.json` config
- `Strategy/` — positioning, pricing, accelerator prep

---

## Making Changes

Edit any note directly in Obsidian. Changes are just markdown files on disk, so your normal git workflow applies.

### Committing and Pushing

```bash
# Check what you've changed
git status

# Stage your changes
git add .

# Commit with a descriptive message
git commit -m "docs: add pricing analysis notes"

# Push to your branch
git push
```

### Working on a Branch

Please don't push directly to `main`. Create a branch for your work:

```bash
# Create and switch to a new branch
git checkout -b your-name/what-youre-doing

# Example
git checkout -b connor/mcp-architecture-notes
```

When your changes are ready, open a PR on GitHub.

### Pulling Latest Changes

```bash
git pull origin main
```

If you're on a branch and want to sync with main:

```bash
git fetch origin
git rebase origin/main
```

---

## Notes

- All files are plain markdown — no lock-in, no proprietary formats
- Don't commit the `.obsidian/workspace.json` file (it's gitignored) — it tracks your local window state and will conflict
- If you install new community plugins locally, let the team know so we can keep the vault config in sync
