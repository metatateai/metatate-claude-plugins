# Security

## Supported Versions

Security fixes are applied to the latest released version of the marketplace and
plugin.

## Reporting A Vulnerability

Do not open a public GitHub issue for suspected credential exposure,
authorization bypass, tenant isolation problems, or other sensitive findings.

Report security issues through your Metatate support or security contact. If
you do not have a private contact, email the Metatate team through the contact
channel listed on the Metatate website and request a private security intake.

## Credential Handling

Snowflake OAuth client secrets (`metatate-snow`):

- Do not commit Snowflake OAuth client secrets.
- Do not paste OAuth client secrets into shell commands.
- Use `claude mcp add-json --client-secret` so Claude Code prompts for the
  secret.
- Rotate the Snowflake OAuth client secret if it appears in shell history,
  screenshots, logs, issue trackers, or chat tools.

Metatate Cloud access tokens (`metatate`):

- Tokens are shown exactly once at issuance in the workspace Tokens tab; only
  a hash is stored server-side.
- Do not commit tokens and do not pass them as command-line arguments. The
  `metatate-cloud-mcp-add` helper deliberately has no token flag: in `--run`
  mode it reads `METATATE_MCP_TOKEN` or a hidden prompt, and in print mode it
  only ever prints the `mtt_<your-access-token>` placeholder.
- The registered token lives in the user's local Claude configuration — the
  same trust boundary as the Snowflake OAuth client secret. During `--run`
  the token is briefly visible in the local process arguments of the
  generated `claude` command.
- Revoke the token in the workspace Tokens tab if it appears in shell
  history, screenshots, logs, issue trackers, or chat tools, then issue a new
  one.

## Data Handling

The plugins are designed for metadata, policy context, intended-use context,
query text, and decision records. They should not request raw row-level data
by default. If your organization enables workflows that inspect raw data,
treat that as a separate governance and audit decision.
