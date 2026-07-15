# Validated Results for Rule 100101

The normalized synthetic JSONL corpus documents the intended field combinations. Final validation was performed end to end through the monitored Windows endpoint because generic JSON replay through `wazuh-logtest` did not reproduce the `windows_eventchannel` decoder path.

| Test | Sanitized target path | Creating image | Observed final rule | Status |
|---|---|---|---|---|
| 1. Expected policy test | `C:\Users\<user>\AppData\Local\Temp\__PSScriptPolicyTest_<id>.ps1` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` | `100101`, level 3 | Passed |
| 2. Matching filename outside the exact user Temp path | `C:\Windows\Temp\__PSScriptPolicyTest_NEGPATH_<id>.ps1` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` | `92201`, level 9 | Passed |
| 3. Unrelated Temp script | `C:\Users\<user>\AppData\Local\Temp\Wazuh_NEGFILENAME_<id>.ps1` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` | `92213`, level 15 | Passed |
| 4. Wrong creating process | `C:\Users\<user>\AppData\Local\Temp\__PSScriptPolicyTest_NEGIMAGE_<id>.ps1` | `C:\Windows\System32\cmd.exe` | `92213`, level 15 | Passed |

## Acceptance Criteria

The rule passed because:

- The positive event decoded as Sysmon Event ID 11 and matched rule `100101` at level 3.
- All three near-miss events decoded through `windows_eventchannel` but did not match rule `100101`.
- The rule required the expected Event ID, exact user Temp path pattern, policy-test filename pattern, and Windows PowerShell 5.1 image.
- No XML, PCRE2, decoder, manager-startup, or rule-loading errors were observed.

The JSONL file is a normalized fixture for field and regex review. It is not a replayable substitute for the live EventChannel validation recorded above.

See [`../../VALIDATION.md`](../../VALIDATION.md) for the sanitized end-to-end evidence matrix and limitations.
