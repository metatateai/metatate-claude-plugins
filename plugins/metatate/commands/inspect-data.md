---
description: Explain governed data meaning, classification, sensitivity, PII, and masking facts for a table or column.
---

# Inspect Governed Data

Use the `metatate` MCP tool `inspect_data_meaning`.

Pass the asset as `ref`: `{"database": ..., "schema": ..., "table": ...}` with
normalized lowercase names. This tool takes `ref`, not `asset`. If the user
mentions one column, add `"column"`; otherwise omit it to inspect the table.

The output is a plain object with no answer state: `meaning`, `data_type`,
`classification` (sensitivity, category, subcategory), `pii`, and `masking`
(type plus exempt roles). Summarize those facts; an unknown asset is an
`asset_not_found` error.
