---
description: Inspect active governance, usage, validation, and transfer rules for a governed asset.
---

# Inspect Governance Rules

Use the `metatate` MCP tool `inspect_governance_rules`.

Pass the asset as `asset`: `{"database": ..., "schema": ..., "table": ...}`,
adding `"column"` when the user names one. Add `scenario_key` only when a
canonical key is already known from `discover_context` output or the user.

Summarize the returned instruction rules, conditions, obligations, and policy
provenance. On `review_required`, present every conflicting source — never
pick a winner. On `not_enough_published_state`, relay `reason_code`,
`unresolved_targets`, and `next_actions`; do not guess.
