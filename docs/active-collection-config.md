# Active Telemetry Collection Configuration

## Purpose

This document records the sanitized configuration elements that enabled the telemetry demonstrated in the lab. It intentionally includes only the minimum reproducible excerpts and excludes credentials, enrollment data, private addressing, host-specific identifiers, and unrelated default configuration.

## Linux Endpoint

### Authentication and system-log collection

The Linux endpoint collects the systemd journal through Wazuh Logcollector:

```xml
<localfile>
  <log_format>journald</log_format>
  <location>journald</location>
</localfile>
```

This source supplied the SSH and PAM records used during live validation of parent rule `5760` and custom correlation rule `100100`. The endpoint does not rely on a separately configured `/var/log/auth.log` `<localfile>` entry in the inspected local configuration.

Additional local sources were present for Wazuh active-response logging and package-management logging:

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/ossec/logs/active-responses.log</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/dpkg.log</location>
</localfile>
```

Unrelated command-monitoring blocks are intentionally omitted from this public excerpt.

### File Integrity Monitoring

File Integrity Monitoring was enabled with startup scanning and a 12-hour scheduled scan:

```xml
<syscheck>
  <disabled>no</disabled>
  <frequency>43200</frequency>
  <scan_on_start>yes</scan_on_start>

  <directories check_all="yes" report_changes="yes" realtime="yes">/root/wazuh-fim-test</directories>
  <directories>/etc,/usr/bin,/usr/sbin</directories>
  <directories>/bin,/sbin,/boot</directories>

  <skip_nfs>yes</skip_nfs>
  <skip_dev>yes</skip_dev>
  <skip_proc>yes</skip_proc>
  <skip_sys>yes</skip_sys>
</syscheck>
```

The custom lab directory used realtime monitoring, complete verification checks, and change reporting. Standard system paths remained under scheduled FIM coverage. The complete default ignore list is omitted because it is lengthy and not necessary to reproduce the demonstrated custom-directory deletion alert.

### Central configuration assignment

The Linux endpoint was assigned to the Wazuh `default` group. The manager-side shared `agent.conf` contained only the empty `<agent_config>` wrapper and a placeholder comment:

```xml
<agent_config>
  <!-- Shared agent configuration here -->
</agent_config>
```

Therefore, no additional centralized Logcollector or FIM settings were applied through `agent.conf` at the time of inspection. The other files in the shared `default` directory were CIS, rootcheck, and audit reference files used by Wazuh components; they were not extra `<localfile>` or `<syscheck>` configuration blocks for this endpoint.

## Windows Endpoint

### Security EventChannel collection

The Windows Security log was collected through the `eventchannel` format with an exclusion query that suppresses selected high-volume event IDs:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
  <query>Event/System[EventID != 5145 and EventID != 5156 and EventID != 5447 and EventID != 4656 and EventID != 4658 and EventID != 4663 and EventID != 4660 and EventID != 4670 and EventID != 4690 and EventID != 4703 and EventID != 4907 and EventID != 5152 and EventID != 5157]</query>
</localfile>
```

This source supplied the authentication, privileged-logon, and process-creation telemetry used in the Windows investigation scenario. The query is preserved exactly as observed so the documented evidence boundary is reproducible.

### Sysmon Operational collection

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

This channel supplied Sysmon process and file-creation telemetry, including the Event ID 11 events used to validate rule `100101`.

### PowerShell collection

Both the modern PowerShell Operational channel and the classic Windows PowerShell log were collected:

```xml
<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Windows PowerShell</location>
  <log_format>eventchannel</log_format>
</localfile>
```

The Operational channel supplied the Event ID 4104 script-block evidence shown in the repository.

### PowerShell Script Block Logging policy

The Local Machine policy enabled both Script Block Logging and Script Block Invocation Logging:

```text
Scope: Local Machine
EnableScriptBlockLogging: 1
EnableScriptBlockInvocationLogging: 1
```

The equivalent Current User policy key was not present. The `Microsoft-Windows-PowerShell/Operational` channel was enabled, used circular retention, and had a configured maximum size of `15728640` bytes. The changing event-record count is intentionally not published because it is runtime state rather than configuration.

### Sysmon configuration fingerprint

The active Sysmon installation reported version `15.21`. Rather than publishing the complete compiled ruleset, the lab records a fingerprint of the `SysmonDrv` registry `Rules` value:

```text
ServiceKey: SysmonDrv
RulesBytes: 435224
RulesSHA256: c1cfa5346efba8f7f7be4ad4405c9dd0cdf9feac84d7ebd4c260842a31fa2fcf
```

The fingerprint identifies the exact compiled Sysmon rules state present during collection without exposing the underlying filters, exclusions, internal paths, or rule contents. Any change to the compiled rules should produce a different SHA-256 value.

### Additional standard Windows logs

The local Wazuh configuration also collected the standard Application and System logs:

```xml
<localfile>
  <location>Application</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>System</location>
  <log_format>eventchannel</log_format>
</localfile>
```

These standard channels are recorded for completeness but were not the primary evidence sources for the two documented investigation scenarios.

### Central configuration assignment

The Windows endpoint was assigned to the Wazuh `default` group. The manager-side shared `agent.conf` was the same empty wrapper inspected for the Linux endpoint, so no additional centralized EventChannel blocks were applied at the time of inspection.

## Sanitization Notes

- Private addresses and account names are not included.
- Full configuration files and the complete Sysmon ruleset are not published.
- Only telemetry-relevant excerpts are retained.
- The custom FIM path is a dedicated lab path and contains no personal username.
- Dynamic event counts are omitted from configuration evidence.
- The Sysmon SHA-256 value is a deliberately reviewed configuration fingerprint, not a credential or enrollment secret.