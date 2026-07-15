# Rule 100100 SSH Correlation Validation

## Purpose

This document records the completed live validation of custom rule `100100` through the normal Linux SSH collection pipeline.

The rule is intended to correlate three alerts from parent rule `5760` when they share the same decoded source IP and occur within 120 seconds.

```xml
<rule id="100100" level="10" frequency="3" timeframe="120">
  <if_matched_sid>5760</if_matched_sid>
  <same_srcip />
  <description>Custom rule: Possible SSH brute force from $(srcip) - 3 failed SSH logins within 120 seconds</description>
</rule>
```

## Safety and Sanitization

Testing used a dedicated valid, non-privileged account and deliberately incorrect passwords. Real account names, complete internal addresses, passwords, and unsanitized authentication logs are not published.

Source relationships are represented consistently as `<source-A>` and `<source-B>`.

## Preliminary Findings

### Invalid-user path

A nonexistent username did not exercise parent rule `5760`. It followed built-in rule `5710`, which matches `illegal user` or `invalid user` activity.

One interactive invalid-user session also produced several related rule `5710` records, so that method was unsuitable for validating a three-event threshold.

### Valid-user path

The installed Wazuh SSH ruleset showed that rule `5760` matches `Failed password`, `Failed keyboard`, or `authentication error` under the SSH decoder path.

The completed correlation tests therefore used a valid non-privileged account with an intentionally incorrect password and one password prompt per SSH session.

## Event Generation Pattern

From a Windows workstation with OpenSSH installed:

```powershell
ssh -o PreferredAuthentications=password `
    -o PubkeyAuthentication=no `
    -o KbdInteractiveAuthentication=no `
    -o PasswordAuthentication=yes `
    -o NumberOfPasswordPrompts=1 `
    -o ConnectTimeout=10 `
    <valid-lab-user>@<linux-endpoint-ip>
```

For the second source, the equivalent command was run from another authorized host with a different LAN address.

Each command was allowed to close after one deliberately incorrect password.

## Evidence Query

The following pattern was used on the Wazuh manager after each test phase:

```bash
sudo tail -n 20000 /var/ossec/logs/alerts/alerts.json |
jq -Rrc '
  fromjson?
  | select(
      (((.rule.id? // "") | tostring) == "5760") or
      (((.rule.id? // "") | tostring) == "100100")
    )
  | {
      timestamp: .timestamp,
      agent: .agent.name,
      rule_id: .rule.id,
      rule_level: .rule.level,
      rule_frequency: .rule.frequency,
      srcip: (.data.srcip // .srcip),
      decoder: .decoder.name
    }
'
```

## Completed Test Matrix

| Case | Controlled input | Observed result | Status |
|---|---|---|---|
| Baseline | One failed password against a valid non-privileged account | Rule `5760`, level 5; decoder `sshd`; decoded source IP present; no rule `100100` | Passed |
| Below threshold | Two failures from `<source-A>` within 120 seconds | Two rule `5760` alerts approximately 14 seconds apart; no rule `100100` | Passed |
| Threshold reached | Three failures from `<source-A>` within 120 seconds | First two events remained rule `5760`; third event triggered rule `100100`, level 10, with `rule.frequency: 3` | Passed |
| Different sources | One failure from `<source-A>` and two from `<source-B>` | All three remained rule `5760`; no rule `100100` | Passed |
| Window reset | Two failures from `<source-A>`, wait more than 120 seconds, then one more failure | All three remained rule `5760`; third event occurred approximately 170 seconds after the second; no rule `100100` | Passed |

## Acceptance Criteria

Rule `100100` passed all acceptance criteria:

1. A single valid-user password failure matched parent rule `5760` and included a decoded source IP.
2. Two same-source failures did not trigger rule `100100`.
3. The third same-source failure within 120 seconds triggered rule `100100` at level 10.
4. Three failures split between two source addresses did not trigger rule `100100`.
5. Events separated by more than 120 seconds did not correlate across the expired window.
6. No decoder, rule-loading, or manager error was observed during the tests.

## Operational Note

The threshold of three failures within 120 seconds is intentionally sensitive for a small lab. A production deployment should tune frequency, timeframe, suppression behavior, and response actions according to exposure, expected login behavior, account-lockout policy, and acceptable alert volume.

See [`../../VALIDATION.md`](../../VALIDATION.md) for the consolidated validation record.