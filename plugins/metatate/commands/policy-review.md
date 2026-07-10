---
description: Review governed policy coverage and likely gaps for a data asset, query, workflow, or intended use.
---

# Metatate Policy Review

Use local repository context to identify candidate data assets or SQL when
needed, then use Metatate MCP tools for governed facts.

Preferred sequence:

1. `discover_context` for unclear assets (each asset lists its
   `scenario_keys`).
2. `get_decision_context` (`asset`, optional `scenario_key`) for each likely
   governed table.
3. `inspect_data_meaning` and `inspect_governance_rules` for sensitive assets.
4. `authorize_use` or `validate_query_context` when the user asks if work can
   proceed.

`not_enough_published_state` answers ARE the gap findings — report their
`reason_code` and `unresolved_targets` as missing coverage, not as failures.
Return existing coverage, gaps, questions for governance, and recommended next
review steps. Do not create or deploy policy.
