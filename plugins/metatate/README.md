# Metatate for Claude Code

This Claude Code plugin adds Metatate governance workflows that use the
Metatate Cloud MCP server — the hosted, cross-platform Metatate workspace.

> Running the Metatate Snowflake Native App instead? Install the
> `metatate-snow` plugin — see
> [`plugins/metatate-snow/README.md`](../metatate-snow/README.md).

No Metatate workspace yet?
[Create a free account](https://app.getmetatate.com/sign-up?ref=claude-plugins)
and load the AcmeCloud demo from the dashboard's "New here?" banner to get a
governed workspace in minutes.

## What It Does

The plugin adds Claude Code commands for:

- Discovering governed data context and canonical scenario keys.
- Inspecting data meaning and active governance rules.
- Asking Metatate whether a proposed data use is allowed.
- Validating SQL or query context before execution or release.
- Explaining authorization decisions from their `decision_id`.
- Running advisory policy reviews and release gates.

## Install

Install from the public marketplace:

```text
/plugin marketplace add metatateai/metatate-claude-plugins
/plugin install metatate@metatate-claude-plugins
```

## Connect The MCP Server

The plugin supplies commands and skills. The MCP connection must be
registered separately in Claude Code.

You need two values from your workspace's MCP page
(`https://<workspace-host>/<workspace>/mcp`):

- the MCP endpoint URL, from the Connect tab
- an access token, from the Tokens tab (admin/owner only; shown exactly once;
  revocable)

Register the endpoint with the token (this matches the snippet the Connect
tab renders):

```bash
claude mcp add-json --scope user metatate '{"type":"http","url":"<mcp-server-url>/mcp","headers":{"Authorization":"Bearer <your-access-token>"}}'
```

There is no OAuth flow: the bearer token is the whole handshake. Treat it like
a password — never commit it or paste it into shared logs, and revoke it in
the Tokens tab if it is ever exposed.

If you cloned the marketplace repository, you can register without putting the
token in shell history:

```bash
./plugins/metatate/bin/metatate-cloud-mcp-add \
  --url https://<workspace-mcp-host> \
  --config-scope user \
  --run
```

`--run` reads the token from `METATATE_MCP_TOKEN` or a hidden prompt.

## Commands

- `/metatate:discover-context`
- `/metatate:inspect-data`
- `/metatate:inspect-rules`
- `/metatate:authorize-use`
- `/metatate:validate-query`
- `/metatate:explain-decision`
- `/metatate:policy-review`
- `/metatate:release-gate`

The plugin expects the MCP server to expose the Metatate Cloud tools:

- `discover_context`
- `get_decision_context`
- `inspect_data_meaning`
- `inspect_governance_rules`
- `authorize_use`
- `validate_query_context`
- `explain_why`

For full setup instructions, see the marketplace README and docs:

- `docs/metatate-cloud-install.md`
- `docs/troubleshooting.md`
