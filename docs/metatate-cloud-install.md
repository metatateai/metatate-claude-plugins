# Claude Code Install Guide (Metatate Cloud)

This guide is for users of **Metatate Cloud** — the hosted Metatate workspace.
Using the Snowflake Native App instead? See
[claude-code-install.md](claude-code-install.md).

## Values You Need

From your workspace's MCP module at
`https://<workspace-host>/<workspace>/mcp`:

- The MCP endpoint URL, shown on the **Connect** tab.
- An access token, issued on the **Tokens** tab. Issuing tokens requires the
  workspace admin or owner role; the token is shown exactly once, can carry an
  optional expiry, and can be revoked at any time.

The token looks like `mtt_` followed by 64 hex characters. Treat it like a
password: never commit it, never paste it into shared logs or chat, and revoke
it in the Tokens tab if it is ever exposed.

## 1. Install The Plugin

Open Claude Code and add the marketplace:

```text
/plugin marketplace add metatateai/metatate-claude-plugins
```

Install the plugin:

```text
/plugin install metatate@metatate-claude-plugins
```

Restart Claude Code if prompted.

## 2. Register The MCP Server

If you cloned the marketplace repository locally, register without putting the
token in shell history:

```bash
./plugins/metatate/bin/metatate-cloud-mcp-add \
  --url https://<workspace-mcp-host> \
  --config-scope user \
  --run
```

`--run` reads the token from the `METATATE_MCP_TOKEN` environment variable, or
prompts for it with input hidden. The helper accepts the base URL or the full
`/mcp` endpoint and normalizes either.

If you do not have a local clone, register the equivalent MCP config directly
— this is the same snippet your workspace's Connect tab renders, with your
endpoint and token filled in:

```bash
claude mcp add-json --scope user metatate '{"type":"http","url":"<mcp-server-url>/mcp","headers":{"Authorization":"Bearer <your-access-token>"}}'
```

Note that pasting a real token into a terminal command records it in shell
history; prefer the helper's `--run` mode.

## 3. Confirm The Connection

There is no OAuth step — the bearer token is the whole handshake. Open Claude
Code and check the MCP menu:

```text
/mcp
```

The `metatate` server should show as connected immediately. To check the
endpoint itself, the health route needs no token:

```bash
curl -sS https://<workspace-mcp-host>/health
```

## 4. Confirm It Works

Run:

```text
/metatate:discover-context
```

Example prompt:

```text
Show governed assets I can inspect. If you need to narrow the search, ask me
for a database or schema.
```

Metatate returns governed assets as `database.schema.table` references with
their canonical scenario keys. Pick one asset, then test a decision workflow:

```text
/metatate:authorize-use
```

Example:

```text
Can we use <database>.<schema>.<table> for <your-intended-use>?
```

Claude should call the Metatate Cloud tools and return an advisory decision
with rationale, any obligations or conditions, and a `decision_id`. Feed that
id to `/metatate:explain-decision` to see the decision's evidence.

## Updating

Update the marketplace metadata:

```text
/plugin marketplace update metatate-claude-plugins
```

Then update the plugin from Claude Code's plugin UI, or reinstall it if your
Claude Code version does not expose update actions in the UI.

## Removing

Remove the MCP server:

```bash
claude mcp remove metatate
```

Then uninstall the plugin:

```text
/plugin uninstall metatate@metatate-claude-plugins
```

Finally, revoke the access token in the workspace Tokens tab — removing the
local registration does not invalidate the token.
