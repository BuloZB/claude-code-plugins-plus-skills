# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Claude Code plugins marketplace (270+ plugins, 739 skills). Live at https://claudecodeplugins.io/

**Monorepo structure:** pnpm workspaces (v9.15.9+)
- `plugins/mcp/*` - MCP server plugins (TypeScript, ~2% of plugins)
- `plugins/[category]/*` - AI instruction plugins (Markdown, ~98%)
- `plugins/saas-packs/*-pack` - SaaS skill packs
- `marketplace/` - Astro website (**uses npm, not pnpm**)
- `packages/` - CLI, validator, analytics

**Package manager:** `pnpm` at root, `npm` for marketplace only.

## Essential Commands

```bash
# Before ANY commit (MANDATORY)
pnpm run sync-marketplace           # Regenerate marketplace.json from .extended.json
./scripts/validate-all-plugins.sh   # Full validation
./scripts/quick-test.sh             # Fast validation (~30s)

# Build & test
pnpm install && pnpm build          # Install and build all
pnpm test && pnpm typecheck         # Run tests and type check

# Single MCP plugin
cd plugins/mcp/[name]/ && pnpm build && chmod +x dist/index.js

# Marketplace website
cd marketplace/ && npm run dev      # Dev server at localhost:4321

# Validation
./scripts/validate-all-plugins.sh plugins/[category]/[name]/
python3 scripts/validate-skills-schema.py --verbose  # 2026 skills schema + grading

# Run single test (CLI package)
cd packages/cli && pnpm test -- --grep "pattern"

# Lint all packages
pnpm lint
```

## Two Catalog System (Critical)

| File | Purpose | Edit? |
|------|---------|-------|
| `.claude-plugin/marketplace.extended.json` | Source of truth with extended metadata | ✅ Yes |
| `.claude-plugin/marketplace.json` | CLI-compatible (auto-generated) | ❌ Never |

Run `pnpm run sync-marketplace` after editing `.extended.json`. CI fails if out of sync.

## Data Flow

```
marketplace.extended.json (source of truth, edit this)
        ↓ pnpm run sync-marketplace
marketplace.json (auto-generated, never edit)
        ↓ CI deploys to Firebase
claudecodeplugins.io/catalog.json
        ↓
ccpi CLI fetches and caches locally
```

Key files:
- `scripts/sync-marketplace.cjs` - Strips extended fields (featured, mcpTools, pricing)
- `marketplace/scripts/discover-skills.mjs` - Extracts skills from plugins

## Plugin Structure

### AI Instruction Plugins
```
plugins/[category]/[plugin-name]/
├── .claude-plugin/plugin.json    # Required: name, version, description, author
├── README.md
├── commands/*.md                 # Slash commands
├── agents/*.md                   # Custom agents
└── skills/[skill-name]/SKILL.md  # Auto-activating skills
```

### MCP Server Plugins
```
plugins/mcp/[plugin-name]/
├── .claude-plugin/plugin.json
├── src/*.ts                      # TypeScript source
├── dist/index.js                 # Must be executable with shebang
├── package.json
└── .mcp.json
```

### SKILL.md Frontmatter (2026 Spec)
```yaml
---
name: skill-name
description: |
  When to use this skill. Include trigger phrases.
allowed-tools: Read, Write, Edit, Bash(npm:*), Glob
version: 1.0.0
author: Name <email>
---
```

Valid tools: `Read`, `Write`, `Edit`, `Bash`, `Glob`, `Grep`, `WebFetch`, `WebSearch`, `Task`, `TodoWrite`, `NotebookEdit`, `AskUserQuestion`, `Skill`

## Adding a New Plugin

1. Copy from `templates/` (minimal, command, agent, or full)
2. Create `.claude-plugin/plugin.json` with required fields
3. Add entry to `.claude-plugin/marketplace.extended.json`
4. `pnpm run sync-marketplace`
5. `./scripts/validate-all-plugins.sh plugins/[category]/[name]/`

## CI Validation

PRs are blocked if any check fails:
- **JSON validation** - All plugin.json files must be valid with required fields
- **Catalog sync** - `marketplace.json` must match regenerated from `.extended.json`
- **Skills schema** - SKILL.md frontmatter must conform to 2026 spec
- **TypeScript** - `pnpm typecheck` must pass
- **Executability** - MCP dist/index.js must have shebang and be executable

## Conventions

- **Hooks:** Use `${CLAUDE_PLUGIN_ROOT}` for portability
- **Scripts:** All `.sh` files must be `chmod +x`
- **Model IDs in skills:** Use `sonnet` or `haiku` (not `opus`)
- **README Contributors:** Newest contributors go at the TOP of the list

## Key Identifiers

- **Slug:** `claude-code-plugins-plus`
- **Install:** `/plugin marketplace add jeremylongshore/claude-code-plugins`

## Task Tracking (Beads)

**Session start (after compaction):**
```bash
bd sync && bd ready              # What's available?
bd list --status in_progress     # What was I working on?
```

**During work:**
```bash
bd update <id> --status in_progress  # Claim task BEFORE starting
# ... do work ...
bd close <id> --reason "..."         # Complete with evidence
```

**Session end:**
```bash
bd sync && git push              # MANDATORY - work isn't done until pushed
```

See `AGENTS.md` for full protocol.
