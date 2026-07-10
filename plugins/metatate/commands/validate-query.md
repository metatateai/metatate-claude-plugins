---
description: Validate SQL or query context against active Metatate governance before execution or release.
---

# Validate Query Context

Use the `metatate` MCP tool `validate_query_context`.

Pass query text as `sql`. Supply `default_database` and `default_schema` when
the SQL has unqualified table names. Pass the intent — a known canonical
`scenario_key`, or the user's purpose as free-text `use` — whenever one is
stated: without intent, only always-applicable read controls and rules on
columns the query references participate in the verdict. Add the transfer
context (`operation`, `destination`, `consumer_jurisdiction`) when the query
feeds an export.

Report the `pass` / `warn` / `fail` verdict and the per-ref findings with
their instructions and `decision_id`s. A finding with `status: "evaluated"`
and `decision: null` means nothing published applies to that query's shape or
intent. On `query_unresolved_identifier`, ask for the defaults instead of
rewriting the SQL.
