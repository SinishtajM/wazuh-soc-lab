# Sanitized Wazuh Alert Examples

## Purpose

This directory contains compact JSON examples derived from alerts observed in the active Wazuh lab. The examples show the field structure used during investigation and custom-rule validation without publishing complete raw alerts or environment-specific identifiers.

These files are evidence excerpts, not synthetic sample events. Each example was created from a real indexed alert, reduced to the fields needed to explain the detection, and reviewed for privacy before publication.

## Published examples

| File | Source alert | Intended value | Status |
|---|---|---|---|
| [`windows-privileged-logon.json`](windows-privileged-logon.json) | Windows Security Event ID `4672`, rule `67028` | Shows the Windows EventChannel provider, rule metadata, account placeholders, logon identifier placeholder, and assigned privilege list | Validated and published |
| [`sysmon-policy-test-tuning.json`](sysmon-policy-test-tuning.json) | Custom rule `100101`, Sysmon Event ID `11` | Shows the exact field structure used by the PowerShell policy-test tuning rule | Validated and published |
| [`linux-ssh-authentication-failure.json`](linux-ssh-authentication-failure.json) | Parent rule `5760` | Shows the decoded SSH source and destination-user fields used by the correlation rule | Validated and published |
| [`ssh-same-source-correlation.json`](ssh-same-source-correlation.json) | Custom rule `100100` | Shows the correlated rule, severity, frequency, and same-source context | Validated and published |

## Publication schema

Each example retains only fields that materially explain the alert:

```json
{
  "timestamp": "<timestamp>",
  "agent": {
    "id": "<agent-id>",
    "name": "<agent-name>"
  },
  "rule": {
    "id": "<rule-id>",
    "level": 0,
    "description": "<sanitized-description>",
    "groups": ["<relevant-group>"]
  },
  "decoder": {
    "name": "<decoder>"
  },
  "data": {
    "<relevant-field>": "<sanitized-value>"
  }
}
```

The published examples omit non-applicable sections and may include an additional rule field such as `frequency` when it is present in the source alert.

## Required sanitization

Environment-specific values use stable placeholders:

- Endpoint names: `<windows-agent>` or `<linux-agent>`
- Agent IDs: `<windows-agent-id>` or `<linux-agent-id>`
- Usernames and account names: `<user>` or `<account>`
- Hostnames and domains: `<endpoint>` or `<domain>`
- Source and destination addresses: `<source-address>` or `<destination-address>`
- File paths containing user or host identifiers: only the identifying segment is replaced
- Event timestamps: `<timestamp>` unless chronology is essential
- Process IDs, GUIDs, record IDs, and other runtime identifiers: omitted unless essential, otherwise replaced with a descriptive placeholder

Sanitization must cover values embedded inside retained strings as well as dedicated fields. For example, the live description for rule `100100` contained the source address; the published description replaces that embedded value with `<source-address>` in addition to sanitizing `data.srcip`.

## Fields that must not be published

Do not include:

- `full_log`
- Complete Windows event XML or rendered message text
- Passwords, keys, tokens, certificates, enrollment material, or authentication hashes
- Unredacted command lines or script-block contents
- Complete internal addressing
- Local usernames, workstation names, domains, or user-profile paths
- Wazuh configuration hashes or shared-file hashes
- Index names, document IDs, tenant IDs, or saved-object metadata

## Validation completed

Each published file passed the following checks:

1. The source alert was confirmed in the active Wazuh deployment.
2. Retained fields were compared with the live decoded alert.
3. Nonessential fields were removed.
4. Environment-specific identifiers were replaced with documented placeholders.
5. JSON syntax was validated with `jq empty`.
6. The result was reviewed for private addresses, usernames, endpoint names, user-profile paths, and opaque runtime identifiers.
7. Rule descriptions were reviewed separately for embedded identifiers.
8. The source rule and event type were documented without retaining the original document ID or exact timestamp.

## Evidence boundary

These examples demonstrate decoded field names and relationships for selected alerts. They do not represent complete alert exports, reconstruct the full original event, or prove that every possible field was present across all alerts of the same type. Repeated group entries and source formatting are retained when they were present in the observed alert because they form part of the decoded evidence structure.
