# Troubleshooting

Sections: [Metatate Cloud](#metatate-cloud) · [Snowflake Native App](#snowflake-native-app) · [Either platform](#either-platform)

## Metatate Cloud

### Every Call Returns `unauthorized`

The server returns the same `unauthorized` error whether the token is missing,
malformed, expired, revoked, or simply wrong — it never says which.

Fix:

1. Check the registration points at the right endpoint:

   ```bash
   claude mcp get metatate
   ```

   The URL must end in `/mcp` and match the endpoint shown on your workspace's
   Connect tab (staging and production workspaces have different hosts).

2. Mint a new token in the workspace's MCP → Tokens tab (admin or owner role)
   and re-register:

   ```bash
   ./plugins/metatate/bin/metatate-cloud-mcp-add --url https://<workspace-mcp-host> --config-scope user --run
   ```

### `auth_unavailable` (503)

Infrastructure-side, not your token. Retry later; if it persists, check the
endpoint health (no token needed):

```bash
curl -sS https://<workspace-mcp-host>/health
```

### `rate_limited` (429)

The per-token rate limit was hit. Wait for the `Retry-After` interval (60
seconds) and retry once. If multiple agents or teammates share one token,
issue a separate token per seat in the Tokens tab.

### `quota_exceeded`

The workspace's metered MCP-call quota for the billing period is exhausted —
every tool call meters one call. This is not fixable client-side; review the
workspace plan and usage.

### Claude Calls The Wrong Tool Names Or Argument Shapes

Metatate Cloud lists snake_case tools (`discover_context`, `authorize_use`)
that take structured references like
`{"database": ..., "schema": ..., "table": ...}`. Hyphenated names and flat
`table_name` strings belong to the Snowflake-managed server and the
`metatate-snow` plugin.

Fix: for Cloud workspaces install `metatate@metatate-claude-plugins` (not
`metatate-snow`), and keep only one of the two plugins installed.

### Both Plugins Installed And The Server Name Collides

Both platforms register their MCP server as `metatate` by default. If you
really need both connections on one machine, give one a different name:

```bash
./plugins/metatate/bin/metatate-cloud-mcp-add --url https://<workspace-mcp-host> --name metatate-cloud --config-scope user --run
```

Claude matches tools, not server names, so the commands keep working.

## Snowflake Native App

### The Role ALL Is Blocked

Error:

```text
The role ALL requested has been explicitly blocked for use with this application by an administrator.
```

Cause:

Snowflake received an OAuth request for the user's default role or secondary
role `ALL` instead of the role intended for Metatate.

Fix:

1. Remove the existing Claude MCP registration.

   ```bash
   claude mcp remove metatate
   ```

2. Register again with an explicit Snowflake role scope.

   ```bash
   claude mcp add-json --scope user --client-secret metatate '{
     "type": "http",
     "url": "https://<account-url>/api/v2/databases/METATATE_APP/schemas/CORE/mcp-servers/METATATE_MCP",
     "oauth": {
       "clientId": "<snowflake-oauth-client-id>",
       "callbackPort": 8080,
       "scopes": "session:role:<snowflake-role>"
     }
   }'
   ```

3. Ask your Snowflake administrator to confirm the same role is allowed and
   preauthorized on the OAuth security integration.

Users should not need to change their default Snowflake role to fix this.

### Browser Login Completes But Claude Still Shows Disconnected

Check:

- The callback URI in Snowflake is exactly
  `http://localhost:8080/callback`.
- The Claude MCP registration uses `"callbackPort": 8080`.
- No other local process is occupying port `8080` during authentication.
- The browser completed the redirect back to localhost.

If your team uses a different callback port, update both Snowflake and Claude.

### Claude Cannot Find The Metatate MCP Tools

Check the MCP registration:

```bash
claude mcp get metatate
```

The URL should look like:

```text
https://<account-url>/api/v2/databases/METATATE_APP/schemas/CORE/mcp-servers/METATATE_MCP
```

Ask your administrator to confirm:

```sql
SHOW MCP SERVERS IN SCHEMA METATATE_APP.CORE;
```

Expected tools:

- `discover-context`
- `get-decision-context`
- `inspect-data-meaning`
- `inspect-governance-rules`
- `authorize-use`
- `validate-query-context`
- `explain-why`

### OAuth Client Secret Was Pasted Into Shell History

Rotate the Snowflake OAuth client secret and remove the exposed value from shell
history according to your organization's security process.

Register MCP again using:

```bash
claude mcp add-json --scope user --client-secret metatate '<json-config>'
```

The `--client-secret` flag makes Claude Code prompt for the secret instead of
requiring it in the command.

## Either Platform

### Plugin Installed But Slash Commands Are Missing

Check:

```text
/plugin
```

Confirm the plugin you installed — `metatate@metatate-claude-plugins`
(Metatate Cloud) or `metatate-snow@metatate-claude-plugins` (Snowflake) — is
installed and enabled.

Then check:

```text
/help
```

If the commands still do not appear, restart Claude Code and update the
marketplace:

```text
/plugin marketplace update metatate-claude-plugins
```

### Permission Or Policy Result Looks Unexpected

Metatate is the source of truth for governance decisions. Capture:

- Actor role (workspace member or Snowflake role).
- Table or asset name.
- Operation and intended use.
- Decision ID or validation ID returned by Metatate.
- Claude prompt used.

Then review the decision with `/metatate:explain-decision` (Metatate Cloud) or
`/metatate-snow:explain-decision` (Snowflake).
