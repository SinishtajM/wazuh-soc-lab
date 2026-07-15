# Validation Record

## Purpose

This file documents what the public repository proves, which behaviors were observed during the original lab, and which items were rerun or collected directly from the active deployment. It prevents later documentation improvements from being mistaken for tests that were not actually performed.

## Validation Status

| Capability | Original lab result | Public evidence | Current validation status |
|---|---|---|---|
| Windows agent connected | Observed | `wazuh-agents-overview.png` | Wazuh `4.14.6`, active; version and registration verified |
| Linux agent connected | Observed | `wazuh-agents-overview.png` | Wazuh `4.14.6`, active; version and registration verified |
| Windows Security events | Observed | Authentication screenshot and investigation narrative | Security EventChannel source and exclusion query verified |
| PowerShell Event ID 4104 | Observed | `windows-powershell-4104.png` | Operational channel, Script Block Logging, and Invocation Logging verified |
| Sysmon telemetry | Observed | `windows-sysmon-activity.png` | Sysmon `15.21`, Operational channel, Event ID 11 path, and rules fingerprint verified |
| Failed SSH authentication | Observed | `linux-ssh-sudo-activity.png` | Live rule `5760` and correlation behavior revalidated |
| sudo/PAM activity | Observed | `linux-ssh-sudo-activity.png` | Linux `journald` collection source verified |
| File deletion through FIM | Observed | `linux-fim-file-deleted.png` | Realtime custom directory and scheduled FIM configuration verified |
| Rule 100100 fired | Observed | `custom-wazuh-rules.png` | Live positive and negative correlation tests passed |
| Rule 100101 fired | Observed | `custom-wazuh-rules.png` | Live positive and three negative tests passed |
| Environment inventory | Not part of original screenshots | `docs/environment-inventory.md` | Exact versions and service states collected and sanitized |
| Active collection configuration | Not part of original screenshots | `docs/active-collection-config.md` | Linux and Windows collection sources verified and sanitized |
| Reusable dashboard queries | Added during documentation improvement | `docs/dashboard-queries.md` and `docs/dashboard-query-validation.md` | Ten DQL searches parsed and returned correctly scoped live results; sample-window gaps documented |
| Logical architecture diagram | Added during documentation improvement | `docs/architecture.md` and the README Mermaid diagram | Verified components and logical flows documented; GitHub rendering reviewed |
| Sanitized decoded-alert examples | Added during documentation improvement | `examples/alerts/` | Four live-derived JSON excerpts validated and privacy reviewed |
| Normalized PowerShell fixtures | Added during documentation improvement | `rules/tests/powershell-policy-test-events.jsonl` | Four synthetic field-review cases aligned with the live validation matrix |
| Automated repository audit | Added during final audit remediation | `.github/workflows/repository-audit.yml` | JSON, XML, links, PNG metadata, address, retired-value, and common secret-pattern checks enabled |

## Evidence Classes

### Observed lab behavior

Events and alerts reviewed during the original controlled exercises.

### Published evidence

Sanitized screenshots, reduced decoded-alert excerpts, normalized field-review fixtures, investigation summaries, architecture documentation, configuration excerpts, exact version inventory, reusable DQL searches, test inputs, and end-to-end validation matrices included in this repository.

### Live revalidation

Controlled Windows and Linux events rerun through the active Wazuh agent pipelines, direct collection of active software versions and telemetry-source configuration, and execution of the published DQL searches through the active Wazuh dashboard.

### Documentation derived from verified evidence

The logical architecture diagram was produced from the collected service inventory, active collection configuration, dashboard-query validation, and end-to-end rule tests. It documents supported component relationships and flow direction; it is not itself a packet capture, physical network survey, port-listener audit, or transport-configuration audit.

The JSON alert examples were reduced from live Wazuh alerts after rule and field behavior had been verified. They retain only the fields needed to demonstrate selected decoded relationships and are not complete raw-event exports.

The PowerShell JSONL corpus is a normalized synthetic fixture based on the revalidated field structure. It supports field and PCRE2 review but is not presented as a replayable substitute for the live Windows EventChannel tests.

### Documentation status

The major portfolio documentation milestones are complete. Future work is limited to maintenance, such as refreshing versions or configuration records when the live lab changes.

## Architecture Documentation Validation

The sanitized logical architecture is maintained in [`docs/architecture.md`](docs/architecture.md) and is rendered directly in the main README.

### Verified diagram scope

| Diagram element | Supporting evidence |
|---|---|
| All-in-one Wazuh server boundary | Manager, indexer, dashboard, and Filebeat were verified as active services on the same Ubuntu server |
| Windows endpoint to manager telemetry flow | Active Windows agent registration and Security, Sysmon, and PowerShell EventChannel sources were verified |
| Linux endpoint to manager telemetry flow | Active Linux agent registration, `journald` collection, SSH/sudo/PAM alerts, and FIM configuration were verified |
| Manager to Filebeat to indexer flow | Installed active services and Wazuh component roles documented in the environment inventory |
| Dashboard to indexer investigation flow | Ten DQL searches were executed through Discover against `wazuh-alerts-*` |
| Dashboard to manager status/configuration flow | Dashboard, manager, and agent status behavior were observed and documented |
| Admin workstation test paths | Controlled Windows and SSH/Linux actions were used for live positive and negative validation |

### Rendering and sanitization review

- GitHub successfully rendered the Mermaid diagram in the pull request.
- Component labels and flow directions were reviewed for readability.
- No private IP addresses, usernames, credentials, certificates, enrollment keys, or environment-specific identifiers are included.
- Port numbers and transport settings are intentionally omitted because every listener and connection was not independently audited against configuration.
- The diagram makes no unsupported claim about Internet exposure, reverse proxies, identity providers, email relays, active-response automation, or third-party integrations.
- Dotted arrows are explicitly defined as controlled test activity rather than continuous telemetry transport.

## Sanitized Alert Example Validation

The evidence guide and four excerpts are maintained in [`examples/alerts/`](examples/alerts/).

| Example | Verified source | Retained behavior |
|---|---|---|
| `windows-privileged-logon.json` | Windows Security Event ID `4672`, rule `67028` | EventChannel decoder, Security provider, rule groups, sanitized account fields, and privilege list |
| `sysmon-policy-test-tuning.json` | Custom rule `100101`, Sysmon Event ID `11` | Rule level, decoder, PowerShell image, and sanitized policy-test filename pattern |
| `linux-ssh-authentication-failure.json` | Parent rule `5760` | Level `5`, `sshd` decoder, and sanitized source and destination-account fields |
| `ssh-same-source-correlation.json` | Custom rule `100100` | Level `10`, frequency `3`, `sshd` decoder, and sanitized same-source context |

### Collection and validation procedure

- Each example was selected from `/var/ossec/logs/alerts/alerts.json` on the active Wazuh server.
- `jq` was used to select the target alert and reduce it to the fields necessary to explain the detection.
- Each generated file passed `jq empty` before publication.
- Retained rule IDs, levels, descriptions, groups, decoder names, event IDs, frequency values, and relevant decoded fields were compared with the source alert.
- Windows paths were normalized to standard JSON escaping without changing their meaning.
- Source formatting that reflected the decoded alert was retained, including repeated rule groups and the leading space in the observed `" WEF"` group value.

### Privacy and publication review

- Agent names and IDs, timestamps, users, domains, SIDs, logon identifiers, addresses, filename identifiers, and user-profile segments were replaced with stable placeholders.
- Rule descriptions were reviewed separately because they can embed identifiers that are not obvious from the structured fields.
- The real source address embedded in rule `100100`'s description was replaced with `<source-address>`.
- Complete `full_log` values, Windows event XML or rendered messages, document IDs, index metadata, command lines, script blocks, credentials, enrollment data, and configuration hashes were excluded.
- The excerpts demonstrate selected field names and relationships; they do not reconstruct the original alerts or prove that every possible field exists in every event of the same type.

## Normalized PowerShell Fixture Validation

The four-line JSONL corpus is maintained in [`rules/tests/powershell-policy-test-events.jsonl`](rules/tests/powershell-policy-test-events.jsonl).

- Source endpoint names and usernames were replaced with the clearly synthetic values `lab-windows-endpoint` and `test-user`.
- Source timestamps, event-record IDs, process identifiers, and filename suffixes were replaced with fixed synthetic values.
- Complete rendered Windows event messages were removed to avoid duplicating identifiers already represented in structured fields.
- The negative-path fixture uses `C:\Windows\Temp`, matching the documented live negative test.
- Each line parses as valid JSON.
- The fixture retains only the fields needed to review the custom rule's Event ID, target-path, filename, and creating-image conditions.
- Conclusive validation remains the live EventChannel matrix, because generic JSON replay does not reproduce the parent decoder and rule chain.

## Environment and Configuration Verification

The completed environment record is maintained in [`docs/environment-inventory.md`](docs/environment-inventory.md). Sanitized active collection excerpts are maintained in [`docs/active-collection-config.md`](docs/active-collection-config.md).

### Verified Wazuh server

- Wazuh manager `4.14.6`, runtime revision `rc2`, package `4.14.6-1`
- Wazuh indexer package `4.14.6-1`
- Wazuh dashboard package `4.14.6-1`
- Filebeat package `7.10.2-2`
- Ubuntu `24.04.4 LTS`, kernel `6.8.0-134-generic`, architecture `x86_64`
- Manager, indexer, dashboard, and Filebeat services active during collection

### Verified Linux endpoint

- Wazuh agent `4.14.6`, runtime revision `rc2`, package `4.14.6-1`
- Ubuntu `24.04.4 LTS`, kernel `6.8.0-134-generic`, architecture `x86_64`
- OpenSSH package `1:9.6p1-3ubuntu13.18`
- Wazuh agent and SSH services active
- Local `journald` collection source verified
- FIM enabled with startup scanning, 12-hour scheduled scans, standard system paths, and realtime monitoring of `/root/wazuh-fim-test`
- Assigned to the `default` group; shared `agent.conf` contained no centralized overrides
- Temporary account used for SSH testing removed after validation

### Verified Windows endpoint

- Wazuh agent `4.14.6`; `WazuhSvc` running with automatic startup
- Windows 11 Pro `25H2`, version `10.0.26200`, full build `26200.8875`, 64-bit
- Sysmon `15.21`; `Sysmon64` running with automatic startup
- Windows PowerShell `5.1.26100.8875`, Desktop edition
- PowerShell build `10.0.26100.8875`; CLR `4.0.30319.42000`
- Security, Sysmon Operational, PowerShell Operational, Windows PowerShell, Application, and System EventChannel sources verified
- Local Machine Script Block Logging and Invocation Logging enabled
- PowerShell Operational channel enabled with circular retention and a `15728640`-byte maximum size
- Assigned to the `default` group; shared `agent.conf` contained no centralized EventChannel overrides

### Sysmon configuration fingerprint

The active compiled Sysmon rules were fingerprinted without publishing their contents:

| Field | Observed value |
|---|---|
| Registry service key | `SysmonDrv` |
| Compiled rules size | `435224` bytes |
| SHA-256 | `c1cfa5346efba8f7f7be4ad4405c9dd0cdf9feac84d7ebd4c260842a31fa2fcf` |

The fingerprint identifies the exact compiled rules state observed during collection but does not expose or reconstruct the full Sysmon configuration.

## Dashboard Query Validation

The reusable catalog is maintained in [`docs/dashboard-queries.md`](docs/dashboard-queries.md), and the sanitized live matrix is maintained in [`docs/dashboard-query-validation.md`](docs/dashboard-query-validation.md).

All ten queries parsed and returned correctly scoped results through Discover using the `wazuh-alerts-*` data view and DQL:

| Query group | Verified behavior |
|---|---|
| Endpoint scoping | Returned only the selected endpoint in visible rows |
| Windows authentication | Returned only requested IDs; visible sample contained 4624 and 4672, while 4625 was absent from that window |
| Windows process and PowerShell | Returned only requested IDs; visible sample contained 4688, while 4104 was absent from that window |
| Sysmon process and file activity | Returned only Sysmon Event IDs 1 and 11 |
| Linux investigation timeline | Returned only requested IDs; visible sample contained 5760, 5501, and 5402, while 553 was absent from that window |
| SSH correlation timeline | Returned parent rule `5760` and custom rule `100100`; frequency 3 appeared on the custom alert |
| PowerShell tuning | Returned only rule `100101`, level 3, Sysmon Event ID 11 |
| Policy-test file activity | Returned the tuned alert and relevant untuned built-in rules |
| Custom-rule overview | Returned only rules `100100` and `100101` with expected endpoint-specific fields |
| High-severity triage | Every visible result had `rule.level` 10 or greater |

The live hostnames, changing hit counts, private addresses, account names, paths, command lines, rule descriptions, and script contents were not retained in the public validation record.

## Rule 100100 SSH Correlation Validation

The completed live validation procedure is documented in [`rules/tests/ssh-brute-force-validation.md`](rules/tests/ssh-brute-force-validation.md). Testing used the normal Linux SSH collection pipeline, a dedicated valid non-privileged test account, and sanitized evidence.

| Case | Expected result | Observed result | Status |
|---|---|---|---|
| One failed SSH authentication | Parent rule `5760`; decoded source present; no rule `100100` | Rule `5760`, level 5; decoder `sshd`; decoded source present; no rule `100100` | Passed |
| Two failures from source A within 120 seconds | Parent alerts only; no rule `100100` | Two rule `5760` alerts at level 5 from the same source, approximately 14 seconds apart; no rule `100100` | Passed |
| Third failure from source A within the same window | Rule `100100`, level 10 | First two events remained rule `5760`; third event triggered rule `100100`, level 10, with `rule.frequency: 3`; all occurred within approximately 22 seconds | Passed |
| Three failures split between sources A and B | No rule `100100` | Three rule `5760` alerts: one from `<source-A>` and two from `<source-B>`; no rule `100100` | Passed |
| Two failures, wait more than 120 seconds, then one more | No rule `100100` | All three remained rule `5760`; the third occurred approximately 170 seconds after the second; no rule `100100` | Passed |

### Important rule-path observation

A nonexistent username exercised built-in rule `5710`, not parent rule `5760`. One invalid-user SSH session also produced multiple related rule `5710` records, making it unsuitable for validating a three-event correlation threshold. The completed tests therefore used a valid non-privileged account with intentionally incorrect passwords.

### Result

Rule `100100` behaved as intended in the tested environment:

1. One or two same-source failures remained at the parent-rule severity.
2. The third same-source failure within 120 seconds triggered rule `100100` at level 10.
3. Failures from different source addresses did not correlate.
4. Failures separated by more than 120 seconds did not correlate across the expired window.
5. The decoded source IP was present for every qualifying rule `5760` event.

Environment-specific addresses and usernames are not published. Source relationships are represented with placeholders such as `<source-A>`, `<source-B>`, and `<test-user>`.

## Rule 100101 Live Field Revalidation

A matching indexed alert was revalidated on July 14, 2026. The source alert was generated on July 12, 2026 and decoded as a Sysmon file-creation event.

| Field | Sanitized observed value |
|---|---|
| `win.system.eventID` | `11` |
| `win.eventdata.targetFilename` | `C:\Users\<user>\AppData\Local\Temp\__PSScriptPolicyTest_<id>.ps1` |
| `win.eventdata.image` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| `win.eventdata.user` | `<endpoint>\<user>` |
| `win.eventdata.processGuid` | Present |
| `win.eventdata.processId` | Present |
| `win.eventdata.parentImage` | Not present in this Sysmon Event ID 11 alert |

The validated rule requires all of the following:

1. Sysmon Event ID 11.
2. The complete user `AppData\Local\Temp` path.
3. A filename beginning with `__PSScriptPolicyTest_` and ending in `.ps1`.
4. The observed Windows PowerShell 5.1 executable path.

## Rule 100101 End-to-End Results

Testing was performed through the monitored Windows endpoint because generic JSON replay through `wazuh-logtest` did not reproduce the `windows_eventchannel` decoder path.

| Case | Expected result | Observed result | Status |
|---|---|---|---|
| Expected policy-test file in the user Temp directory, created by Windows PowerShell | Rule `100101`, level 3 | Rule `100101`, level 3; decoder `windows_eventchannel`; Sysmon Event ID 11 | Passed |
| Matching policy-test filename outside the exact user Temp path | Rule `100101` must not match | Built-in rule `92201`, level 9; decoder `windows_eventchannel`; Sysmon Event ID 11 | Passed |
| Unrelated `.ps1` file in the user Temp directory | Rule `100101` must not match | Built-in rule `92213`, level 15; decoder `windows_eventchannel`; Sysmon Event ID 11 | Passed |
| Matching policy-test filename created by a non-PowerShell process | Rule `100101` must not match | Built-in rule `92213`, level 15; decoder `windows_eventchannel`; Sysmon Event ID 11; creating image `cmd.exe` | Passed |

## Automated Repository Audit

The workflow in [`.github/workflows/repository-audit.yml`](.github/workflows/repository-audit.yml) validates the current tracked tree on pushes and pull requests.

It checks:

- all four reduced alert examples and every JSONL line for valid JSON;
- the normalized JSONL corpus contains exactly four events and no rendered `win.system.message` field;
- `rules/local_rules.xml` has valid XML structure when wrapped as a ruleset fragment;
- relative Markdown links resolve to current tracked files or directories;
- PNG files have valid structure and contain no `tEXt`, `zTXt`, `iTXt`/XMP, or `eXIf` chunks;
- UTF-8 text files contain no valid private IPv4 addresses or retired source-derived labels from the original fixture;
- common private-key and token formats are not present in tracked text.

These automated checks reduce regression risk but do not replace manual review of visible screenshot content, rule semantics, or Git history.

## Publication and Sanitization Checks

Completed checks:

- Private IPv4 addresses redacted from screenshots
- Windows and Linux usernames redacted
- Environment-specific paths and identifiers redacted where necessary
- Screenshots flattened to prevent recovery of covered pixels
- Published PNG files validated to contain no `tEXt`, `zTXt`, `iTXt`/XMP, or `eXIf` metadata chunks
- Duplicate `local_rules.xml.txt` artifact removed
- Repository wording changed from RDP-specific language to evidence-supported remote-authentication language
- Source-derived alert IPs, usernames, process identifiers, event-record IDs, and document identifiers excluded or replaced with clearly synthetic fixture values
- Rule `100100` validated with baseline, below-threshold, same-source threshold, different-source, and expired-window cases
- Rule `100101` validated with one positive and three negative cases
- Exact Wazuh, operating-system, agent, Sysmon, and PowerShell versions recorded
- Active Linux and Windows collection sources reviewed and sanitized
- Ten reusable DQL searches live validated through the Wazuh dashboard, with sample-window gaps recorded explicitly
- Live hostnames, hit counts, addresses, usernames, commands, paths, and script contents excluded from the query-validation record
- Saved-object exports deliberately excluded because they may contain data-view IDs, tenant metadata, or substituted local values
- Logical architecture diagram rendered successfully in GitHub and reviewed for readable labels and flow direction
- Architecture diagram excludes private identifiers, credentials, enrollment material, unverified ports, transport settings, and unsupported integrations
- Four live-derived JSON alert excerpts validated with `jq empty` and compared with their source alerts
- PowerShell JSONL fixtures normalized to synthetic endpoint, account, timestamp, filename, process, and event-record values; rendered event messages omitted
- Agent identifiers, account data, addresses, timestamps, runtime identifiers, and user-profile segments replaced in the alert examples
- Rule descriptions reviewed for embedded identifiers before publication
- Complete raw alert bodies, document metadata, commands, scripts, credentials, enrollment data, and configuration hashes excluded from the alert examples
- Wazuh configuration hashes and shared-file hashes excluded
- Sysmon compiled-rules fingerprint deliberately retained as non-secret reproducibility evidence

## Known Limitations

- Screenshots and reduced JSON excerpts are representative and do not show every event or field in each investigation timeline.
- The JSON examples are selected evidence excerpts, not complete sanitized raw-alert exports.
- The normalized JSONL fixture is useful for field and regex review but does not reproduce the live `windows_eventchannel` rule chain.
- Generic JSON replay through `wazuh-logtest` uses the `json` decoder and does not reproduce the `windows_eventchannel` rule chain for rule `100101`.
- Rule `100100` uses an intentionally sensitive lab threshold; production deployments require environment-specific tuning.
- Rule `100101` was validated against Windows PowerShell 5.1; PowerShell 7 requires separate review and testing.
- The published evidence supports remote Windows authentication but does not independently prove that the session used RDP.
- The Sysmon fingerprint identifies compiled state but does not publish the complete underlying ruleset.
- Saved-object exports are not published because they can contain environment-specific metadata even when the visible DQL query is generic.
- The architecture is a logical model derived from verified evidence; it is not a physical network map, packet capture, comprehensive listener or transport audit, or claim of unverified integrations.
- Automated current-tree checks do not remove or certify older binary revisions retained in Git history.

## Next Documentation Milestone

The portfolio documentation is complete for the current lab state. Future updates should focus on periodic version and configuration refreshes or newly implemented detections rather than presentation-only additions.
