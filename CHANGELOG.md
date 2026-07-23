# Changelog

Entries are per plugin from 0.2.0 onward. Versions 0.1.0–0.1.3 describe the
Snowflake plugin (now `metatate-snow`) before the marketplace carried two
plugins.

## metatate 0.2.1

- Replaced the README demo animation with the Metatate Cloud contract:
  snake_case tool names, structured asset references, an `answered`
  CONDITIONAL cross-border decision with a uuid `decision_id`, and publication
  provenance. The previous image predated the 0.2.0 split and showed the
  Snowflake contract under `/metatate:` commands.
- Documented the MCP request log: every tool call is recorded server-side with
  allow-listed fields (asset path, scenario key, answer state, decision —
  never SQL or intended-use text), and workspace admins can review per-token
  history under MCP → Tokens → View requests (README, install guide,
  troubleshooting, and the skill).
- Refreshed the AcmeCloud sample-workspace copy for the two-tier loader:
  starter sample (5 tables, 4 policies, one live publication) and "Load the
  full estate" (9 tables, 15 policies, 103 published instruction rows).
- Taught the scenario-lane model (`platform` / `agent` / `human`, with
  `scenario_lane` echoed in answers) and the agent-lane `ai.*` keys in the
  skill, `policy-review`, `release-gate`, and the example prompts.
- Corrected the troubleshooting guidance: the Cloud server accepts hyphenated
  tool names as 1:1 aliases — Snowflake-shaped flat arguments are what fail
  (`invalid_parameters`).

## metatate 0.2.0

- New plugin for **Metatate Cloud** (the hosted workspace MCP server). The
  eight commands and the `metatate-governance` skill teach the Metatate
  Cloud contract: snake_case tool names (`discover_context`, …),
  structured asset references (`{"database", "schema", "table", "column"}`),
  the typed three-state answer model (`answered` / `review_required` /
  `not_enough_published_state` with stable `reason_code`s), canonical
  scenario-key discipline, transfer context on `authorize_use` and
  `validate_query_context`, intent-aware query validation, and
  `decision_id` → `explain_why` chaining.
- Added `bin/metatate-cloud-mcp-add`, which registers the bearer-token MCP
  connection without putting the access token in files or shell history
  (reads `METATATE_MCP_TOKEN` or a hidden prompt in `--run` mode).
- Added `docs/metatate-cloud-install.md` and Metatate Cloud sections to the
  README, troubleshooting guide, and example prompts.

## metatate-snow 0.2.0

- Renamed the Snowflake Native App plugin from `metatate` to `metatate-snow`
  (directory `plugins/metatate-snow/`, commands `/metatate-snow:*`).
  Workflows, tool contract, helper, and Snowflake OAuth setup are unchanged.

## 0.1.3

- Refined the README opening, feature summary, and plugin metadata to align
  with Metatate's structured context, portable decision layer, and agent
  coordination positioning.

## 0.1.2

- Added the full no-clone `claude mcp add-json` registration command to the
  main README, so marketplace-only installers do not need a local checkout to
  configure MCP.

## 0.1.1

- Added quick access to the Metatate Snowflake Marketplace listing from the
  main README prerequisites.
- Reworked smoke tests and example prompts to discover governed assets first
  and use customer-specific placeholders instead of demo table names.

## 0.1.0

- Initial public Metatate Claude Code plugin marketplace.
- Added governed data discovery, inspection, authorization, query validation,
  decision explanation, policy review, and release gate commands.
- Added Snowflake-managed MCP setup documentation.
- Added OAuth role-scoped MCP registration guidance using
  `session:role:<snowflake-role>`.
- Added GitHub Actions validation for the marketplace, plugin, helper script,
  role-scoped OAuth output, and Markdown docs.
