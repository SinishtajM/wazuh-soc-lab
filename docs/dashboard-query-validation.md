# Dashboard Query Validation Record

## Purpose

This document records live validation results for the reusable DQL searches in [`dashboard-queries.md`](dashboard-queries.md). Results are summarized with generic endpoint labels and exclude private addresses, usernames, incident identifiers, raw event contents, and environment-specific saved-object metadata.

## Validation environment

- Wazuh alerts data view: `wazuh-alerts-*`
- Query language: DQL
- Dashboard view: Discover
- Time ranges are selected locally to include known matching events and are not embedded in the reusable queries.

## Results

All ten reusable queries were executed through the active Wazuh dashboard and passed live validation.

| Query | Live result | Status |
|---|---|---|
| Endpoint alert scope | `agent.name: "<windows-agent>"` parsed successfully and returned a nonzero result set containing only the selected Windows endpoint in the visible rows. Columns `timestamp`, `agent.name`, `rule.id`, `rule.level`, and `rule.description` were available and useful. | Passed |
| Windows authentication chain | `agent.name: "<windows-agent>" and data.win.system.eventID: ("4625" or "4624" or "4672")` parsed successfully and returned a nonzero result set. The visible rows contained Event IDs `4624` and `4672` only; no unrelated Event IDs appeared. Event ID `4625` was not present in the visible sample and may depend on the selected time range. The `data.win.system.eventID` column was available and useful. | Passed |
| Windows process and PowerShell activity | `agent.name: "<windows-agent>" and data.win.system.eventID: ("4688" or "4104")` parsed successfully and returned a nonzero result set. Every visible row contained Event ID `4688`; no unrelated Event IDs appeared. Event ID `4104` was not present in the visible sample and may depend on the selected time range. Safe columns `timestamp`, `agent.name`, `data.win.system.eventID`, `rule.id`, and `rule.level` were available and useful. | Passed |
| Sysmon process and file activity | `agent.name: "<windows-agent>" and rule.groups: "sysmon" and data.win.system.eventID: ("1" or "11")` parsed successfully and returned a nonzero result set. Both Sysmon Event IDs `1` and `11` were present in the visible rows, and no unrelated Event IDs appeared. The custom rule `100101` was visible on one Event ID `11` row. Safe columns `timestamp`, `agent.name`, `data.win.system.eventID`, `rule.id`, and `rule.level` were available and useful. | Passed |
| Linux investigation timeline | `agent.name: "<linux-agent>" and rule.id: ("5760" or "5501" or "5402" or "553")` parsed successfully and returned a nonzero result set. The visible rows contained only rule IDs `5760`, `5501`, and `5402`; no unrelated rule IDs appeared. Rule `553` was not present in the visible sample and may depend on the selected time range. Safe columns `timestamp`, `agent.name`, `rule.id`, and `rule.level` were available and useful. | Passed |
| SSH authentication and correlation | `agent.name: "<linux-agent>" and rule.id: ("5760" or "100100")` parsed successfully and returned a nonzero result set containing only parent rule `5760` and custom rule `100100`. Parent-rule rows showed no `rule.frequency` value, while custom-rule rows showed `rule.frequency: 3` and level `10`. Safe columns `timestamp`, `agent.name`, `rule.id`, `rule.level`, and `rule.frequency` were available and useful. | Passed |
| PowerShell policy-test tuning | `agent.name: "<windows-agent>" and rule.id: "100101"` parsed successfully and returned a nonzero result set. Every visible row was custom rule `100101`, level `3`, with Sysmon Event ID `11`; no built-in rule IDs or unrelated event IDs appeared. Safe columns `timestamp`, `agent.name`, `rule.id`, `rule.level`, and `data.win.system.eventID` were available and useful. | Passed |
| PowerShell policy-test file activity | `agent.name: "<windows-agent>" and data.win.system.eventID: "11" and data.win.eventdata.targetFilename: *__PSScriptPolicyTest_*` parsed successfully and returned a nonzero result set. Every visible row was Sysmon Event ID `11`. The results included custom rule `100101` and built-in rules including `92213` and `92205`, confirming that the broader filename query returns tuned and untuned policy-test activity. Safe columns `timestamp`, `agent.name`, `rule.id`, `rule.level`, and `data.win.system.eventID` were available and useful. | Passed |
| Custom-rule overview | `rule.id: ("100100" or "100101")` parsed successfully and returned only the two custom rules. Rule `100100` appeared on the Linux endpoint at level `10` with `rule.frequency: 3` and no Windows event ID. Rule `100101` appeared on the Windows endpoint at level `3` with Sysmon Event ID `11` and no frequency value. The blank non-applicable fields behaved as expected. | Passed |
| High-severity alerts | `rule.level >= 10` parsed successfully and returned a nonzero result set. Every visible row had a rule level of `10` or greater. Results included both Windows and Linux alerts, including custom rule `100100` at level `10`; custom rule `100101` was correctly absent because it is level `3`. Safe columns `timestamp`, `agent.name`, `rule.id`, and `rule.level` were available and useful. | Passed |

## Publication notes

The live endpoint name, changing hit counts, private addresses, account names, workstation names, full rule descriptions, command lines, file paths, and script-block contents are intentionally not retained. The validation record proves parsing and field behavior without publishing local identifiers or runtime-specific dashboard state.