---
description: Run an advisory Metatate governance review for a repository change before merge or release.
---

# Metatate Release Gate

Review local diffs, SQL, dbt models, migrations, notebooks, or application code
paths to identify governed data references. Use Metatate MCP tools for policy
facts and decisions.

Preferred sequence:

1. Identify changed files and candidate SQL/data assets.
2. Use `discover_context` and `get_decision_context` for assets.
3. Use `validate_query_context` for SQL queries or generated query paths —
   pass the change's intent (`scenario_key` or `use`) so intent-scoped rules
   participate in the verdict. For AI-related changes (RAG, training,
   embeddings, agent workflows), use the matching agent-lane `ai.*` key from
   the asset's `scenario_keys` as that intent.
4. Use `authorize_use` for explicit intended-use questions, with transfer
   context for exports.
5. Use `explain_why` with a returned `decision_id` when a decision needs
   evidence.

Map answers to the gate: `answered` with allow or `pass` → pass;
`review_required` or a `warn` verdict → warn; deny, `fail`, or
`not_enough_published_state` on a load-bearing asset → block. The result is
advisory — report it with `decision_id` evidence and
`publication.publication_id`, and recommend concrete fixes. Do not mutate
policy or deploy changes.
