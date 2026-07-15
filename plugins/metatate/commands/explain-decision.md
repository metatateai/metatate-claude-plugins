---
description: Explain a prior Metatate authorization decision.
---

# Explain Metatate Decision

Use the `metatate` MCP tool `explain_why`.

Pass `{"kind": "decision", "decision_id": "<uuid>"}`. The `decision_id` comes
from a prior tool result — `authorize_use` returns the winning `decision_id`
and every cited instruction record carries its own — or from the workspace
decision views. Never fabricate one.

Query-validation explanations are not supported on Metatate Cloud yet; when
the user asks to explain a validation, say so instead of calling the tool.

Summarize the explanation, the cited instruction record (policy, version,
scenario, decision reason), whether it belongs to the `current` publication,
and the publication provenance.
