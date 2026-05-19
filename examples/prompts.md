# Example Prompts

Use these examples after the plugin is installed and the `metatate` MCP server
is connected.

## Discover Governed Context

```text
/metatate:discover-context
Show governed assets I can inspect. If you need to narrow the search, ask me
for a database, schema, domain, sensitivity level, or compliance tag.
```

```text
/metatate:discover-context
Find governed assets in database <database_name> and schema <schema_name>.
```

## Inspect Data Meaning

```text
/metatate:inspect-data
Explain the governed meaning of <fully-qualified-governed-table>.
```

```text
/metatate:inspect-data
What does column <column_name> mean in <fully-qualified-governed-table>, and
what sensitivity or PII facts are known?
```

## Inspect Governance Rules

```text
/metatate:inspect-rules
Which rules apply to <fully-qualified-governed-table> for <your-intended-use>?
```

## Authorize Data Use

```text
/metatate:authorize-use
Can role <your-snowflake-role> read <fully-qualified-governed-table> for
<your-intended-use>?
```

```text
/metatate:authorize-use
Can role <your-snowflake-role> export columns <column_list> from
<fully-qualified-governed-table> to <destination-system> in
<destination-jurisdiction> for <your-intended-use>?
```

## Validate Query Context

```text
/metatate:validate-query
Validate this SQL for <your-intended-use> by role <your-snowflake-role>:

select <column_list>
from <fully-qualified-governed-table>
where <business-filter>
limit 10;
```

## Explain A Decision

```text
/metatate:explain-decision
Explain decision DECISION_123 and tell me what policy evidence mattered.
```

## Review A Local Change

```text
/metatate:policy-review
Review the SQL changes in this branch and identify any governed data risks.
```

```text
/metatate:release-gate
Run an advisory Metatate release gate for this PR.
```
