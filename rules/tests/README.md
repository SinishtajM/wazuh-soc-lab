# Rule 100101 Test Corpus

## Purpose

This directory contains sanitized Sysmon Event ID 11 samples used to review the field conditions in the narrowed PowerShell policy-test tuning rule.

The samples are normalized synthetic fixtures based on the decoded field structure revalidated from a live Wazuh alert. They preserve the rule-relevant field combinations without retaining the source endpoint name, username, timestamp, filename identifier, process identifiers, event-record IDs, or rendered event message.

## Test Cases

The file [`powershell-policy-test-events.jsonl`](powershell-policy-test-events.jsonl) contains four JSON events, one per line and in this order:

1. Expected policy-test filename in a user Temp directory, created by Windows PowerShell.
2. Matching policy-test filename under `C:\Windows\Temp`, outside the expected user Temp directory.
3. Unrelated PowerShell script inside the expected user Temp directory.
4. Matching policy-test filename created by `cmd.exe` instead of Windows PowerShell.

See [`expected-results.md`](expected-results.md) for the validated result matrix.

## Important Wazuh Logtest Limitation

The live source alert was processed with location `EventChannel` and decoder `windows_eventchannel`. Feeding the normalized, already-structured JSON samples directly to `wazuh-logtest` selects the generic `json` decoder instead.

The `-l EventChannel` option sets the test location but does not force the Windows EventChannel decoder. In this situation, built-in Sysmon parent rule `92213` is not evaluated, so custom child rule `100101` cannot reach Phase 3. A result with decoder `json` is therefore inconclusive rather than a rule failure.

The JSONL corpus remains useful for:

- verifying that all required fields are present;
- reviewing the positive and negative field combinations;
- testing the PCRE2 expressions independently;
- documenting the intended rule behavior.

## End-to-End Validation Method

Final validation used events generated on the monitored Windows endpoint and delivered through the normal Wazuh agent EventChannel pipeline.

Before activating the candidate rule, the active file was backed up and the ruleset configuration was tested:

```bash
sudo cp -a /var/ossec/etc/rules/local_rules.xml \
  /var/ossec/etc/rules/local_rules.xml.backup-$(date +%Y%m%d-%H%M%S)

sudo /var/ossec/bin/wazuh-analysisd -t
```

After the configuration test returned successfully, the Wazuh manager was restarted and four controlled Sysmon Event ID 11 file-creation cases were generated.

## Validated Results

| Case | Observed final rule | Status |
|---|---|---|
| Expected Temp policy-test file created by Windows PowerShell | `100101`, level 3 | Passed |
| Matching filename under `C:\Windows\Temp` | `92201`, level 9 | Passed |
| Unrelated `.ps1` file inside the user Temp path | `92213`, level 15 | Passed |
| Matching user-Temp filename created by `cmd.exe` | `92213`, level 15 | Passed |

All four live events decoded through `windows_eventchannel` as Sysmon Event ID 11. Only the expected positive case matched rule `100101`.

The sanitized evidence matrix and limitations are recorded in [`../../VALIDATION.md`](../../VALIDATION.md).
