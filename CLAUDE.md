# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code **plugin marketplace** that ships a single plugin, `metatate`, which surfaces Metatate governed data workflows in Claude Code via the Snowflake-managed MCP server installed by the Metatate Snowflake Native App.

There is no application runtime here. The repo is plugin manifests, command/skill markdown, a Bash helper, and docs. The "behavior" lives in (a) what Claude does when it reads the command and skill markdown, and (b) the MCP tools exposed by Snowflake — not in this codebase.

## Validation commands

CI runs in `.github/workflows/validate.yml`. To reproduce locally:

```bash
# Validate marketplace + plugin manifests (needs Claude Code installed)
claude plugin validate .
claude plugin validate plugins/metatate

# Syntax-check the MCP registration helper
bash -n plugins/metatate/bin/metatate-mcp-add

# Smoke-test the helper's role-scoped OAuth output (no Snowflake call made)
plugins/metatate/bin/metatate-mcp-add \
  --account-url https://example.snowflakecomputing.com \
  --client-id TEST_CLIENT_ID \
  --snowflake-role METATATE_CLAUDE_USER \
  --config-scope user
# Expect output containing: `claude mcp add-json` and `session:role:METATATE_CLAUDE_USER`

# Markdown lint (config: .markdownlint.json — MD013/line-length disabled)
npx --yes markdownlint-cli2 "**/*.md"
```

## Architecture

Two-layer split — keep these separate when editing:

1. **Plugin layer (this repo).** Adds slash commands and a skill to Claude Code. Pure prompt-engineering artifacts; no code execution beyond the helper script.
2. **MCP layer (NOT in this repo).** Snowflake's managed MCP server registered by the Metatate Native App, normally `METATATE_APP.CORE.METATATE_MCP`. Users register it themselves via `claude mcp add-json` (or the helper). The plugin assumes — but does not provide — these MCP tools: `discover-context`, `get-decision-context`, `inspect-data-meaning`, `inspect-governance-rules`, `authorize-use`, `validate-query-context`, `explain-why`.

### Plugin contents (`plugins/metatate/`)

- `.claude-plugin/plugin.json` — plugin manifest. `version` here MUST stay aligned with `version` in `.claude-plugin/marketplace.json` at repo root and with the release git tag (e.g. `v0.1.1`).
- `commands/*.md` — one slash command per file. Each is a thin prompt that names the MCP tool to call and the arguments to collect (e.g. `authorize-use.md` → MCP tool `authorize-use`). Frontmatter `description` becomes the command description in Claude Code.
- `skills/metatate-governance/SKILL.md` — the master behavior contract. Documents the full MCP tool signatures (required vs optional args, JSON-string vs CSV conventions) and rules like "treat Metatate as source of truth", "keep local repo analysis separate from Metatate facts", "don't request raw row-level data by default". When editing any command, cross-check against this skill so contracts stay consistent.
- `bin/metatate-mcp-add` — Bash helper that generates the right `claude mcp add` / `claude mcp add-json` invocation. Two output modes depending on whether OAuth scopes are pinned (`--snowflake-role` / `--oauth-scopes`); the role-scoped branch emits `add-json` with a `session:role:<ROLE>` scope, which Snowflake requires so OAuth doesn't fall back to the user's default role or secondary `ALL`. The helper never accepts a client secret on the CLI — Claude Code prompts for it.

### Marketplace root (`.claude-plugin/marketplace.json`)

Lists the plugin(s) the marketplace exposes. Currently one entry pointing at `./plugins/metatate`. Adding a plugin = new entry here + new directory under `plugins/`.

## Editing notes specific to this repo

- **Tool contracts live in `SKILL.md`.** If you change a command's argument set, update both `commands/<name>.md` and the matching bullet in `skills/metatate-governance/SKILL.md`. Drift between them silently misleads Claude at runtime.
- **Slash-command name = filename.** `commands/authorize-use.md` → `/metatate:authorize-use`. The README, plugin README, and `examples/prompts.md` all list the command set; keep them in sync when adding/removing commands.
- **Version bumps are three-way.** Bump `plugins/metatate/.claude-plugin/plugin.json#version`, `.claude-plugin/marketplace.json#plugins[0].version`, and add a CHANGELOG entry; the release tag (`vX.Y.Z`) should match.
- **Never put secrets in command examples or the helper.** The helper deliberately routes the client secret through Claude Code's interactive prompt; README and docs reinforce this. Don't add a `--client-secret <value>` style flag.
- **OAuth redirect URI is load-bearing.** `http://localhost:8080/callback` must match between Snowflake's `OAUTH_REDIRECT_URI` and the helper's `--callback-port` (default 8080). If you change one default, change the docs in lockstep.
