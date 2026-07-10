---
description: Ask Metatate whether a proposed data use is allowed, denied, conditional, or needs review.
---

# Authorize Data Use

Use the `metatate` MCP tool `authorize_use`.

Collect or infer:

- `asset`: `{"database": ..., "schema": ..., "table": ...}` (add `"column"`
  when the question is column-specific)
- `use`: the user's intended use, verbatim free text
- optional `scenario_key`: only a canonical key already seen in
  `discover_context` output — never invent one
- transfer context whenever data leaves the platform (export, send, share):
  `operation`, `destination` (`{"system": ..., "jurisdiction": ...}`), and
  `consumer_jurisdiction` as short identifier tokens, so transfer rules
  answer destination-specifically. For these transfer questions also pass the
  asset's transfer scenario key (usually `residency.cross_border_transfer`)
  as `scenario_key` — free text like "export" can map to a different
  scenario and miss the transfer rules

Report the decision, `decision_id`, reason, obligations, conditions
(approval, anonymize-first, role), prohibitions, `next_actions`,
`can_proceed_now`, and publication provenance. Offer
`/metatate:explain-decision` on the returned `decision_id`. On
`scenario_unresolved`, fetch the asset's `scenario_keys` via
`discover_context` and ask the user to pick — do not retry with a guessed
key. The answer is advisory; never present it as enforcement.
