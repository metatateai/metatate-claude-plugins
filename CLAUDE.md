# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code **plugin marketplace** that ships two plugins surfacing Metatate governed data workflows in Claude Code:

| Plugin | Platform | MCP layer it targets |
| --- | --- | --- |
| `metatate` (`plugins/metatate/`) | Metatate Cloud (hosted workspace) | Metatate Cloud MCP server, `POST <mcp-server-url>/mcp`, bearer `mtt_` access token |
| `metatate-snow` (`plugins/metatate-snow/`) | Snowflake Native App | Snowflake-managed MCP server, normally `METATATE_APP.CORE.METATATE_MCP`, Snowflake OAuth |

There is no application runtime here. The repo is plugin manifests, command/skill markdown, two Bash helpers, and docs. The "behavior" lives in (a) what Claude does when it reads the command and skill markdown, and (b) the MCP tools exposed by the target server — not in this codebase.

Users install exactly ONE of the two plugins. Before 0.2.0 the `metatate` name belonged to the Snowflake plugin; the README's migration callout and the CHANGELOG cover that history — do not remove them casually.

## Validation commands

CI runs in `.github/workflows/validate.yml`. To reproduce locally:

```bash
# Validate marketplace + plugin manifests (needs Claude Code installed)
claude plugin validate .
claude plugin validate plugins/metatate
claude plugin validate plugins/metatate-snow

# Syntax-check the MCP registration helpers
bash -n plugins/metatate/bin/metatate-cloud-mcp-add
bash -n plugins/metatate-snow/bin/metatate-mcp-add

# Smoke-test the Snowflake helper's role-scoped OAuth output (no Snowflake call made)
plugins/metatate-snow/bin/metatate-mcp-add \
  --account-url https://example.snowflakecomputing.com \
  --client-id TEST_CLIENT_ID \
  --snowflake-role METATATE_CLAUDE_USER \
  --config-scope user
# Expect output containing: `claude mcp add-json` and `session:role:METATATE_CLAUDE_USER`

# Smoke-test the Cloud helper's print mode (no token may ever be printed)
METATATE_MCP_TOKEN=mtt_0000000000000000000000000000000000000000000000000000000000000000 \
plugins/metatate/bin/metatate-cloud-mcp-add --url https://mcp.example.test --config-scope user
# Expect: `claude mcp add-json`, a URL ending in /mcp, the mtt_<your-access-token>
# placeholder — and NEVER the value of METATATE_MCP_TOKEN.

# Markdown lint (config: .markdownlint.json — MD013/line-length disabled)
npx --yes markdownlint-cli2 "**/*.md"
```

## Architecture

Two-layer split — keep these separate when editing:

1. **Plugin layer (this repo).** Adds slash commands and a skill to Claude Code. Pure prompt-engineering artifacts; no code execution beyond the helper scripts.
2. **MCP layer (NOT in this repo).** One server per platform, registered by the user:
   - **Metatate Cloud:** stateless streamable-HTTP MCP at `POST <mcp-server-url>/mcp`; auth is a workspace-issued bearer token (`Authorization: Bearer mtt_…`); tools are snake_case (`discover_context`, `get_decision_context`, `inspect_data_meaning`, `inspect_governance_rules`, `authorize_use`, `validate_query_context`, `explain_why`) with structured snake_case inputs and the typed three-state answer model (`answered` / `review_required` / `not_enough_published_state`).
   - **Snowflake:** Snowflake's managed MCP server registered by the Metatate Native App; tools are hyphenated (`discover-context`, …) with flat scalar arguments (`table_name`, `columns_csv`, `sql_text`) and a `{status, data, errors}` envelope.

### Plugin contents (both `plugins/metatate/` and `plugins/metatate-snow/`)

- `.claude-plugin/plugin.json` — plugin manifest. `version` here MUST stay aligned with that plugin's entry in `.claude-plugin/marketplace.json` at repo root and with the plugin's release git tag.
- `commands/*.md` — one slash command per file, same eight basenames in both plugins. Each is a thin prompt that names the MCP tool to call and the arguments to collect. Frontmatter `description` becomes the command description in Claude Code.
- `skills/metatate-governance/SKILL.md` — the master behavior contract for that platform. Documents the full MCP tool signatures and rules like "treat Metatate as source of truth". When editing any command, cross-check against the SAME PLUGIN's skill so contracts stay consistent — and never port argument shapes across plugins: the platforms' contracts differ by design (structured `asset`/`ref` objects vs flat `table_name`; `use` free text vs `operation`/`intended_use`; typed answer states vs status envelope).
- `bin/` — the registration helper. `metatate-cloud-mcp-add` (Cloud) takes `--url` and reads the access token from `METATATE_MCP_TOKEN` or a hidden prompt in `--run` mode; print mode only ever prints the `mtt_<your-access-token>` placeholder. `metatate-mcp-add` (Snowflake) generates the `claude mcp add`/`add-json` invocation with Snowflake OAuth; the role-scoped branch emits a `session:role:<ROLE>` scope, which Snowflake requires so OAuth doesn't fall back to the user's default role or secondary `ALL`.

### Marketplace root (`.claude-plugin/marketplace.json`)

Lists both plugins. Plugin names must be unique within the marketplace (validation rejects duplicates). Adding a plugin = new entry here + new directory under `plugins/`.

## Editing notes specific to this repo

- **Tool contracts live in each plugin's `SKILL.md`.** If you change a command's argument set, update both `commands/<name>.md` and the matching bullet in that plugin's `skills/metatate-governance/SKILL.md`. Drift between them silently misleads Claude at runtime.
- **The Cloud contract's source of truth is the metatate-saas repo** (`packages/mcp-core/src/schemas/*`, `docs/policy-model-mcp-tools-product-contract.md`, ADR-0009/0010/0011). The web app's Connect-tab snippet (`apps/web/lib/mcp/connection.ts`) is canonical — keep the `claude mcp add-json` snippets in this repo byte-consistent with it.
- **Slash-command name = filename, namespaced by plugin.** `plugins/metatate/commands/authorize-use.md` → `/metatate:authorize-use`; `plugins/metatate-snow/commands/authorize-use.md` → `/metatate-snow:authorize-use`. The README, plugin READMEs, and `examples/prompts.md` all list both command sets; keep them in sync when adding/removing commands.
- **Version bumps are per-plugin and four-way.** Bump `plugins/<name>/.claude-plugin/plugin.json#version`, the matching `.claude-plugin/marketplace.json` entry, add a CHANGELOG section under that plugin's heading, and tag `metatate-vX.Y.Z` or `metatate-snow-vX.Y.Z`. (Tags `v0.1.0`–`v0.1.3` predate the split.)
- **Never put secrets in command examples or the helpers.** The Snowflake helper routes the client secret through Claude Code's interactive prompt — don't add a `--client-secret <value>` style flag. The Cloud helper must never gain a `--token <value>` flag, must never print a real token in print mode, and reads the token only from the env var or a hidden prompt in `--run` mode.
- **The MCP server alias stays `metatate` on BOTH platforms** (helper defaults and all docs). Command bodies referring to "the `metatate` MCP tool …" mean the server alias, not the plugin name — do not rename those references when touching the Snowflake plugin.
- **OAuth redirect URI is load-bearing (Snowflake).** `http://localhost:8080/callback` must match between Snowflake's `OAUTH_REDIRECT_URI` and the helper's `--callback-port` (default 8080). If you change one default, change the docs in lockstep.
