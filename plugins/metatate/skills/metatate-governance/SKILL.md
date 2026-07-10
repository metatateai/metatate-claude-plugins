---
name: metatate-governance
description: Use when a user asks about Metatate governed data context, data meaning, policy rules, data-use authorization, query validation, decision explanation, or release readiness on Metatate Cloud (the hosted workspace MCP server).
---

# Metatate Governance (Metatate Cloud)

Use this skill when a user asks about governed data context, data meaning,
policy rules, data-use authorization, query validation, decision explanation,
or release readiness against a Metatate Cloud workspace.

## Which Server

These rules cover the Metatate Cloud MCP server, normally registered as
`metatate` (tools appear as `mcp__metatate__<tool>`). It lists snake_case tool
names (`discover_context`) and takes structured asset references. The Metatate
Snowflake Native App plugin (`metatate-snow`) covers the Snowflake-managed
server instead, which lists hyphenated tool names (`discover-context`) and
takes flat `table_name` strings — do not mix the two contracts.

If the connected server's `tools/list` schemas differ from this document, the
live schemas win.

## Wire Conventions

- Every input and output key is snake_case. There are no camelCase aliases.
- Assets are structured references using the catalog's normalized lowercase
  names — `{"database": ..., "schema": ..., "table": ..., "column": ...}`
  (`column` optional) — never dotted `db.schema.table` strings.
- `inspect_data_meaning` takes the reference as `ref`; every other
  asset-taking tool takes it as `asset`.
- One exception: cited records' `parameters` carry the authored policy DSL
  verbatim, with camelCase keys such as `destinationSystems`.

## Tool Contract

- `discover_context`: optional `database`, `schema`. (`domain`, `sensitivity`,
  `pii`, and `compliance_tag` are accepted by the schema but answer
  `not_supported` today.) Answers list governed assets with
  `instruction_count` and canonical `scenario_keys`.
- `get_decision_context`: `asset` and/or `proposed_use`, optional
  `scenario_key`; `proposed_use` alone answers `not_supported`. Answers rank
  every applicable instruction, give `effective_decision`, and include
  structured `business_context` (owner, steward, domain, purpose) when
  published.
- `inspect_data_meaning`: required `ref`. Plain-object output with no answer
  state: meaning, data type, classification, PII, and masking facts.
- `inspect_governance_rules`: required `asset`, optional `scenario_key`.
- `authorize_use`: required `asset` and free-text `use` (max 2000 chars);
  optional `scenario_key`; optional transfer context — `operation`,
  `destination` (`{"system": ..., "jurisdiction": ...}`, at least one field),
  and `consumer_jurisdiction` — as short identifier tokens (max 64 chars).
  With transfer context, transfer rules are evaluated
  destination-specifically (authored order, first match wins; unmatched
  context falls to the authored default). A destination-resolved transfer
  answer can return `decision: "conditional"` with the specific `conditions`
  (approval, anonymize-first).
- `validate_query_context`: required `sql` (max 100k chars); optional
  `default_database`, `default_schema`, an intent (`scenario_key` or `use`),
  and the same transfer context. Without intent, only always-applicable read
  controls and rules on columns the query references participate;
  `SELECT *` pulls column rules in.
- `explain_why`: `{"kind": "decision", "decision_id": "<uuid>"}`.
  `{"kind": "validation", ...}` answers `not_supported`.

All tools are read-only and advisory. They never approve, publish, mask,
block, or write anything.

## Answer States

Except `inspect_data_meaning`, every tool answers one of three states:

- `answered`: current published instruction rows are sufficient. Summarize the
  decision or verdict, the cited instructions, and publication provenance.
- `review_required` (`reason_code`: `decision_requires_review` or
  `conflicted_published_state`): present every conflicting source from
  `conflict.sources` — never rank or hide one.
- `not_enough_published_state` (`reason_code`: `no_current_publication`,
  `no_published_instruction_state`, `unresolved_published_target`, or
  `scenario_unresolved`): relay `reason_code`, `unresolved_targets`, and
  `next_actions` verbatim and stop. Missing state is a typed answer, not an
  error and not a decision.

## Scenario And Intent Discipline

- Scenario keys are canonical and dotted (for example `purpose.prohibited_use`,
  `ai.training`, `residency.cross_border_transfer`). Never invent one. Valid
  sources: `discover_context` output (`scenario_keys`), keys echoed in prior
  answers, or the user.
- Without a known key, pass the user's purpose as free-text `use`; the server
  maps it deterministically (explicit key first, then use mapping, then the
  `residency.cross_border_transfer` default when destination context is
  present) and never guesses.
- On `scenario_unresolved`, fetch the asset's `scenario_keys` and ask the user
  to pick.
- For transfer questions, pass the asset's transfer scenario key (usually
  `residency.cross_border_transfer`) explicitly alongside the destination
  context. The automatic destination fallback applies only when the use text
  maps to nothing — words like "export" map to `export.external` and can miss
  the transfer rules.
- Pass intent to `validate_query_context` whenever the user states a purpose,
  and transfer context whenever data leaves (export, send, share, train
  elsewhere).

## Decision IDs

- `authorize_use` answers include the winning `decision_id`, and every cited
  instruction record carries its own `decision_id`. Any of them can be
  explained with `explain_why {"kind": "decision"}`.
- Include `decision_id` values and `publication.publication_id` in
  user-facing summaries when they are returned. Never fabricate a uuid.

## Errors

Closed error codes with fixed messages:

- `unauthorized` (uniform 401): the token is missing, expired, revoked, or
  wrong — the server never says which. Direct the user to mint a new token in
  the workspace MCP page (Tokens tab) and re-register the server.
- `auth_unavailable` (503): infrastructure, not the token. Retry later.
- `rate_limited` (429): per-token limit. Wait `Retry-After` seconds and retry
  at most once; never hammer.
- `quota_exceeded`: the workspace's metered MCP-call quota is exhausted
  (every tool call meters one call). Stop calling and point the user at their
  plan and usage.
- `query_unresolved_identifier`: supply `default_database` and
  `default_schema` instead of rewriting the SQL.
- `not_supported`, `asset_not_found`, `invalid_parameters`, `read_failed`,
  `query_parse_failed`, `output_validation_failed`, `internal_error`: report
  the code plainly and adjust inputs where possible.

## Behavior

- Treat Metatate as the source of truth for policy, catalog, authorization,
  validation, and explanation.
- Do not infer authorization from policy text alone when `authorize_use` or
  `validate_query_context` can answer.
- Keep local repository analysis separate from Metatate facts. Use local files
  to identify candidate tables, SQL, dbt models, migrations, and code paths,
  then call Metatate tools for governed context.
- Do not request or expose raw row-level data by default.
- Every answer is advisory; never present one as enforcement.
