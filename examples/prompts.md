# Example Prompts

Use these examples after a plugin is installed and its `metatate` MCP server
is connected. The first section is for the Metatate Cloud plugin (`metatate`),
the second for the Snowflake Native App plugin (`metatate-snow`).

## Metatate Cloud

Asset references on Metatate Cloud are structured lowercase identifiers. The
examples below use the AcmeCloud demo domain that ships with the public
Metatate examples (`acmecloud_demo.public.customers` and friends) — replace
them with your own governed assets.

### Discover Governed Context

```text
/metatate:discover-context
Show governed assets I can inspect, with their scenario keys.
```

```text
/metatate:discover-context
Find governed assets in database acmecloud_demo, schema public.
```

### Inspect Data Meaning

```text
/metatate:inspect-data
Explain the governed meaning of acmecloud_demo.public.customers.
```

```text
/metatate:inspect-data
What does the email column mean in acmecloud_demo.public.customers, and what
sensitivity, PII, or masking facts are known?
```

### Inspect Governance Rules

```text
/metatate:inspect-rules
Which rules apply to acmecloud_demo.public.customers?
```

### Authorize Data Use

```text
/metatate:authorize-use
Can we use acmecloud_demo.public.customers for renewal-risk analytics
reporting?
```

```text
/metatate:authorize-use
Can we use acmecloud_demo.public.customers for a marketing email campaign?
```

Expect a deny here when the AcmeCloud prohibited-use policy is published.

```text
/metatate:authorize-use
Can we export acmecloud_demo.public.customers to SALESFORCE in the US for
consumers in the EU?
```

A destination-aware transfer rule answers this: SALESFORCE typically returns
conditional with approval and anonymize-first conditions, while ADS_PLATFORM
returns deny. Note the `decision_id` in the answer.

### Validate Query Context

```text
/metatate:validate-query
Validate this SQL for renewal-risk reporting:

select region, count(*) as customers, avg(arr) as avg_arr
from acmecloud_demo.public.customers
group by region;
```

```text
/metatate:validate-query
Validate this SQL for a marketing campaign:

select customer_name, email
from customers;

Default database acmecloud_demo, default schema public.
```

The same SQL can pass for one intent and fail for another — always state the
purpose.

### Explain A Decision

```text
/metatate:explain-decision
Explain decision <decision_id-from-a-previous-authorize-answer> and tell me
what policy evidence mattered.
```

### Review A Local Change

```text
/metatate:policy-review
Review the SQL changes in this branch and identify any governed data risks.
```

```text
/metatate:release-gate
Run an advisory Metatate release gate for this PR.
```

## Snowflake Native App

### Discover Governed Context

```text
/metatate-snow:discover-context
Show governed assets I can inspect. If you need to narrow the search, ask me
for a database, schema, domain, sensitivity level, or compliance tag.
```

```text
/metatate-snow:discover-context
Find governed assets in database <database_name> and schema <schema_name>.
```

### Inspect Data Meaning

```text
/metatate-snow:inspect-data
Explain the governed meaning of <fully-qualified-governed-table>.
```

```text
/metatate-snow:inspect-data
What does column <column_name> mean in <fully-qualified-governed-table>, and
what sensitivity or PII facts are known?
```

### Inspect Governance Rules

```text
/metatate-snow:inspect-rules
Which rules apply to <fully-qualified-governed-table> for <your-intended-use>?
```

### Authorize Data Use

```text
/metatate-snow:authorize-use
Can role <your-snowflake-role> read <fully-qualified-governed-table> for
<your-intended-use>?
```

```text
/metatate-snow:authorize-use
Can role <your-snowflake-role> export columns <column_list> from
<fully-qualified-governed-table> to <destination-system> in
<destination-jurisdiction> for <your-intended-use>?
```

### Validate Query Context

```text
/metatate-snow:validate-query
Validate this SQL for <your-intended-use> by role <your-snowflake-role>:

select <column_list>
from <fully-qualified-governed-table>
where <business-filter>
limit 10;
```

### Explain A Decision

```text
/metatate-snow:explain-decision
Explain decision DECISION_123 and tell me what policy evidence mattered.
```

### Review A Local Change

```text
/metatate-snow:policy-review
Review the SQL changes in this branch and identify any governed data risks.
```

```text
/metatate-snow:release-gate
Run an advisory Metatate release gate for this PR.
```
