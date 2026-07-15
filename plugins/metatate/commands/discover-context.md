---
description: Find governed Metatate assets and their canonical scenario keys by database or schema.
---

# Discover Governed Context

Use the `metatate` MCP tool `discover_context`.

Optional arguments: `database`, `schema` (normalized lowercase identifiers).
Do not send `domain`, `sensitivity`, `pii`, or `compliance_tag` — Metatate
Cloud answers `not_supported` for those filters today; say so when the user
asks to filter by them.

An `answered` result lists governed assets with `ref`
(`{database, schema, table, column}`), `instruction_count`, and
`scenario_keys` — the canonical scenario keys valid for follow-up calls.
Summarize the assets and the best next Metatate inspection step. On
`not_enough_published_state`, relay `reason_code` and `next_actions`
verbatim; do not guess.
