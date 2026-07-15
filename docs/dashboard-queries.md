# Wazuh Dashboard Query Catalog

## Purpose

This document provides reusable Dashboards Query Language (DQL) searches for the investigation and detection workflows demonstrated in this repository. The queries are intended for the Wazuh dashboard Threat Hunting or Discover view using the Wazuh alerts data view.

The catalog uses generic placeholders and does not publish private addresses, local usernames, or environment-specific account names. Replace values such as `<windows-agent>` and `<linux-agent>` only in the local dashboard.

## Current validation status

All ten queries were executed through the active Wazuh dashboard using the `wazuh-alerts-*` data view and DQL. Each query parsed successfully and returned correctly scoped results without unrelated visible event or rule IDs. Some optional target IDs were absent from the selected visible sample windows, as documented in [`dashboard-query-validation.md`](dashboard-query-validation.md).

| Query | Intended evidence | Status |
|---|---|---|
| Endpoint alert scope | All indexed alerts for one selected agent | Passed live dashboard validation |
| Windows authentication chain | Events 4625, 4624, and 4672 | Scope passed; 4625 was not present in the selected visible sample |
| Windows process and PowerShell activity | Events 4688 and 4104 | Scope passed; 4104 was not present in the selected visible sample |
| Sysmon process and file activity | Sysmon Events 1 and 11 | Passed live dashboard validation |
| Linux investigation timeline | SSH, PAM, sudo, and FIM rule IDs | Scope passed; rule 553 was not present in the selected visible sample |
| SSH authentication and correlation | Rules 5760 and 100100 | Passed live dashboard validation |
| PowerShell policy-test tuning | Rule 100101 | Passed live dashboard validation |
| PowerShell policy-test file activity | Tuned and untuned Sysmon Event ID 11 results | Passed live dashboard validation |
| Custom-rule overview | Rules 100100 and 100101 | Passed live dashboard validation |
| High-severity alerts | Rule level 10 or greater | Passed live dashboard validation |

## Dashboard preparation

1. Open the Wazuh dashboard Threat Hunting or Discover view.
2. Select the Wazuh alerts data view, normally `wazuh-alerts-*`.
3. Select DQL as the query language when the interface provides a language selector.
4. Set an appropriate time range before running a query.
5. Replace only the documented placeholders with local values.
6. Review the returned fields and timestamps before saving the query.

OpenSearch Dashboards can save a query together with optional filters and an optional time filter. For reusable investigation searches, save the DQL query but normally leave the time filter excluded so the search can be reused across different periods.

## Core queries

### 1. Scope alerts to one endpoint

**Suggested saved-query name:** `SOC - Selected endpoint alerts`

```text
agent.name: "<agent-name>"
```

Use this as the base query when reviewing all indexed alerts for a single endpoint.

### 2. Windows authentication and privileged-logon chain

**Suggested saved-query name:** `SOC - Windows authentication chain`

```text
agent.name: "<windows-agent>" and data.win.system.eventID: ("4625" or "4624" or "4672")
```

This search is intended to reconstruct failed authentication, successful logon, and special-privilege assignment events for the Windows investigation scenario.

Private investigation columns:

- `timestamp`
- `agent.name`
- `data.win.system.eventID`
- `rule.id`
- `rule.level`
- `rule.description`

### 3. Windows process creation and PowerShell script blocks

**Suggested saved-query name:** `SOC - Windows process and PowerShell`

```text
agent.name: "<windows-agent>" and data.win.system.eventID: ("4688" or "4104")
```

This search is intended to pair Security Event ID 4688 process-creation alerts with PowerShell Operational Event ID 4104 script-block alerts.

Private investigation columns:

- `timestamp`
- `data.win.system.eventID`
- `rule.id`
- `rule.level`
- `rule.description`
- `data.win.eventdata.commandLine`
- `data.win.eventdata.scriptBlockText`

The final two columns should be added only when those fields are present in the selected alerts.

### 4. Sysmon process and file activity

**Suggested saved-query name:** `SOC - Sysmon process and file activity`

```text
agent.name: "<windows-agent>" and rule.groups: "sysmon" and data.win.system.eventID: ("1" or "11")
```

This search is intended to return Sysmon process-creation and file-creation alerts while avoiding unrelated Windows providers that may reuse the same numeric event IDs.

Private investigation columns:

- `timestamp`
- `data.win.system.eventID`
- `rule.id`
- `rule.level`
- `data.win.eventdata.image`
- `data.win.eventdata.parentImage`
- `data.win.eventdata.targetFilename`

Not every Sysmon event type contains every listed field.

### 5. Linux SSH, PAM, sudo, and FIM timeline

**Suggested saved-query name:** `SOC - Linux investigation timeline`

```text
agent.name: "<linux-agent>" and rule.id: ("5760" or "5501" or "5402" or "553")
```

This search reconstructs the Linux scenario using the rule IDs documented in the repository:

- `5760` — failed SSH authentication
- `5501` — PAM session opened
- `5402` — successful sudo to root
- `553` — monitored file deleted

Private investigation columns:

- `timestamp`
- `rule.id`
- `rule.level`
- `rule.description`
- `data.srcip`
- `data.dstuser`
- `full_log`

### 6. SSH failures and custom same-source correlation

**Suggested saved-query name:** `SOC - SSH failures and brute-force correlation`

```text
agent.name: "<linux-agent>" and rule.id: ("5760" or "100100")
```

This query shows the parent SSH authentication failures and the custom same-source threshold alert in one timeline.

Private investigation columns:

- `timestamp`
- `rule.id`
- `rule.level`
- `rule.frequency`
- `data.srcip`
- `data.dstuser`
- `rule.description`

### 7. PowerShell policy-test tuning alert

**Suggested saved-query name:** `SOC - PowerShell policy-test tuning`

```text
agent.name: "<windows-agent>" and rule.id: "100101"
```

This is the narrowest reliable search for the validated custom tuning alert.

Private investigation columns:

- `timestamp`
- `rule.id`
- `rule.level`
- `data.win.system.eventID`
- `data.win.eventdata.image`
- `data.win.eventdata.targetFilename`
- `rule.description`

### 8. PowerShell policy-test file activity, including untuned parent alerts

**Suggested saved-query name:** `SOC - PowerShell policy-test file activity`

```text
agent.name: "<windows-agent>" and data.win.system.eventID: "11" and data.win.eventdata.targetFilename: *__PSScriptPolicyTest_*
```

This broader search is intended to find both the tuned positive case and related Sysmon Event ID 11 alerts that remained on built-in rules during negative testing.

### 9. Custom-rule overview

**Suggested saved-query name:** `SOC - Custom rule alerts`

```text
rule.id: ("100100" or "100101")
```

This query provides a concise view of both custom rules maintained in `rules/local_rules.xml`.

### 10. High-severity alert review

**Suggested saved-query name:** `SOC - High severity alerts`

```text
rule.level >= 10
```

This query returns alerts at Wazuh level 10 or greater. It is useful as a triage starting point but should not replace scenario-specific searches.

## Public-evidence column guidance

When preparing screenshots or public validation evidence, prefer only:

- `timestamp`
- `agent.name`
- `data.win.system.eventID`
- `rule.id`
- `rule.level`
- `rule.frequency` when relevant

Do not display `rule.description`, `full_log`, addresses, usernames, command lines, script-block text, image paths, or target filenames unless each visible value has been individually reviewed and sanitized. Fields useful for private investigation are not automatically safe for publication.

## Optional refinements

After a base query is validated, narrow it with additional DQL clauses rather than publishing real values. Examples:

```text
and data.srcip: "<source-address>"
```

```text
and data.dstuser: "<account>"
```

```text
and rule.level >= 10
```

```text
and decoder.name: "windows_eventchannel"
```

Do not commit the substituted private values to the public repository.

## Saving queries safely

Use descriptive names that identify the workflow without embedding a hostname, username, address, or incident identifier. When saving:

- Include filters only when they are generic and reusable.
- Normally exclude the time filter so the query can be reused.
- Do not export saved-object files without reviewing them for internal data-view IDs, tenant identifiers, private values, or other environment-specific metadata.
- Keep screenshots focused on query behavior and redact local identifiers before publication.

## Validation method

Each query was validated through the active dashboard using a time range containing known matching events. The public validation record retains only:

- Whether the query parsed successfully.
- Whether expected events were returned in the selected visible sample.
- Whether unrelated events were excluded as intended.
- Which fields were useful as visible columns.

Raw results containing private addresses, usernames, host identifiers, paths, commands, or script contents are not published.

## References

- [OpenSearch Dashboards Query Language](https://docs.opensearch.org/latest/dashboards/dql/)
- [OpenSearch Discover search bar and saved queries](https://docs.opensearch.org/latest/dashboards/discover/search-bar/)
