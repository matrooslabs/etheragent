# EtherAgent — Claude Code Plugin Marketplace

## Project Structure

Monorepo using pnpm workspaces + turborepo. Each plugin lives in `packages/plugins/<name>/`.

## Plugin Structure

Every plugin follows this layout:

```
<plugin-name>/
├── package.json           # npm package config with build script
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest (name, version, description, author)
├── skills/
│   └── <skill-name>/
│       └── SKILL.md       # Skill definition with YAML frontmatter
└── scripts/
    └── <skill-name>.ts    # TypeScript script for the skill
```

## Adding a New Plugin

1. Create `packages/plugins/<name>/`
2. Add `package.json` with a `build` script that compiles TypeScript
3. Add `.claude-plugin/plugin.json` with plugin metadata
4. For each skill, create `skills/<skill>/SKILL.md` and `scripts/<skill>.ts`
5. Run `pnpm install && pnpm build` from the repo root

## SKILL.md Format

```yaml
---
description: What the skill does and when to use it
allowed-tools: Bash(git *), Read   # optional tool permissions
---

Skill instructions in markdown. Use $ARGUMENTS for user input.
Use !`command` to run shell commands as preprocessing.
```

## Commands

- `pnpm install` — install all dependencies
- `pnpm build` — build all plugins via turborepo
- `pnpm clean` — remove build artifacts
