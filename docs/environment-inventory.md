# Environment and Version Inventory

## Purpose

This document records the exact software and operating-system versions used by the Wazuh SOC lab. Values were collected directly from the live systems and sanitized before publication.

The inventory intentionally excludes private IP addresses, usernames, hostnames, enrollment keys, certificates, credentials, Wazuh agent configuration hashes, shared-file hashes, keep-alive timestamps, and complete service configuration files. The Sysmon compiled-rules SHA-256 value is intentionally retained as a non-secret configuration fingerprint.

## Wazuh Server

| Component | Version | Collection source | Status |
|---|---|---|---|
| Wazuh manager | `4.14.6` (`WAZUH_REVISION="rc2"`; package `4.14.6-1`) | `wazuh-control info` and package database | Active; verified |
| Wazuh indexer | Package `4.14.6-1` | Package database | Active; verified |
| Wazuh dashboard | Package `4.14.6-1` | Package database | Active; verified |
| Filebeat | Package `7.10.2-2` | Package database | Active; verified |
| Server operating system | Ubuntu `24.04.4 LTS` (Noble Numbat) | `/etc/os-release` | Verified |
| Server kernel | `6.8.0-134-generic` | `uname -r` | Verified |
| Server architecture | `x86_64` | `uname -m` | Verified |

All four Wazuh-related services queried during collection—manager, indexer, dashboard, and Filebeat—reported an `active` systemd state. The runtime manager command reported Wazuh `4.14.6` with revision string `rc2`, while the installed Debian package reported `4.14.6-1`; both values are retained to preserve the exact observed inventory.

## Linux Endpoint

| Component | Version | Collection source | Status |
|---|---|---|---|
| Wazuh agent | `4.14.6` (`WAZUH_REVISION="rc2"`; package `4.14.6-1`) | `wazuh-control info` and package database | Active; verified |
| Ubuntu operating system | Ubuntu `24.04.4 LTS` (Noble Numbat) | `/etc/os-release` | Verified |
| Linux kernel | `6.8.0-134-generic` | `uname -r` | Verified |
| Architecture | `x86_64` | `uname -m` | Verified |
| OpenSSH server | Package `1:9.6p1-3ubuntu13.18` | Package database | Active; verified |

The Linux endpoint reported both the Wazuh agent and SSH services as `active`. The temporary non-privileged account used for SSH correlation testing was removed after validation and is not part of the retained environment.

## Windows Endpoint

| Component | Version | Collection source | Status |
|---|---|---|---|
| Wazuh agent | `4.14.6` | Installed-product metadata and manager registration | `WazuhSvc` running; automatic startup; verified |
| Windows edition | Microsoft Windows 11 Pro | `Win32_OperatingSystem` | Verified |
| Windows display version | `25H2` | Windows current-version registry metadata | Verified |
| Windows version | `10.0.26200` | `Win32_OperatingSystem` | Verified |
| Windows full build | `26200.8875` | Build number plus UBR | Verified |
| Windows architecture | `64-bit` | `Win32_OperatingSystem` | Verified |
| Sysmon | `15.21` | `Sysmon64` service executable metadata | Running; automatic startup; verified |
| Windows PowerShell | `5.1.26100.8875` (`PSEdition: Desktop`) | `$PSVersionTable` | Verified |
| PowerShell build version | `10.0.26100.8875` | `$PSVersionTable.BuildVersion` | Verified |
| .NET CLR used by Windows PowerShell | `4.0.30319.42000` | `$PSVersionTable.CLRVersion` | Verified |
| Sysmon compiled rules size | `435224` bytes | `SysmonDrv` registry `Rules` value | Verified |
| Sysmon compiled rules SHA-256 | `c1cfa5346efba8f7f7be4ad4405c9dd0cdf9feac84d7ebd4c260842a31fa2fcf` | SHA-256 of compiled `Rules` bytes | Verified |

The Wazuh service was reported as `WazuhSvc`, the Sysmon service as `Sysmon64`, and the Sysmon driver registry key as `SysmonDrv`. No executable path, username, private address, or complete Sysmon ruleset is published.

The Sysmon fingerprint identifies the exact compiled rules state observed during collection. It does not reveal the rule contents and will change if the active Sysmon configuration changes.

## Wazuh Agent Registration

| Agent | Wazuh ID | Reported version | Reported operating system | Status |
|---|---:|---|---|---|
| Linux endpoint | `<linux-agent-id>` | Wazuh `4.14.6` | Linux, kernel `6.8.0-134-generic`, `x86_64` | Active; verified |
| Windows endpoint | `<windows-agent-id>` | Wazuh `4.14.6` | Microsoft Windows 11 Pro | Active; verified |

The public inventory uses generic endpoint labels and agent-ID placeholders. Private addresses, local account names, Wazuh configuration hashes, shared-file hashes, and timing metadata from `agent_control` are not recorded.

## Active Collection Configuration Inventory

Sanitized excerpts are maintained in [`active-collection-config.md`](active-collection-config.md).

| Area | Sanitized evidence | Status |
|---|---|---|
| Linux authentication collection | Local `journald` `<localfile>` source | Verified |
| Linux File Integrity Monitoring | Enabled `syscheck`, realtime custom lab path, standard system directories, startup and scheduled scans | Verified |
| Linux agent group assignment | `default` group | Verified |
| Shared `default` group `agent.conf` | Empty `<agent_config>` wrapper; no centralized Logcollector or FIM overrides | Verified |
| Windows agent group assignment | `default` group; same empty shared `agent.conf` | Verified |
| Windows Security channel | Local `Security` EventChannel block with explicit event-ID exclusion query | Verified |
| PowerShell Operational channel | Local `Microsoft-Windows-PowerShell/Operational` EventChannel block | Verified |
| Windows PowerShell log | Local `Windows PowerShell` EventChannel block | Verified |
| Sysmon Operational channel | Local `Microsoft-Windows-Sysmon/Operational` EventChannel block | Verified |
| Standard Windows logs | Local `Application` and `System` EventChannel blocks | Verified |
| PowerShell Script Block Logging | Local Machine policy enabled; invocation logging enabled; Operational channel active | Verified |
| Sysmon configuration | Compiled-rules byte count and SHA-256 fingerprint | Verified |

## Sanitization Requirements

Before publishing any command output or configuration excerpt:

1. Remove private IPv4 and IPv6 addresses.
2. Replace real usernames with placeholders such as `<user>`.
3. Remove host-specific GUIDs, agent keys, certificates, tokens, enrollment data, Wazuh configuration hashes, and shared-file hashes.
4. Remove complete internal paths when they identify a user or private directory structure.
5. Preserve exact product names and version numbers.
6. Preserve configuration elements necessary to reproduce the telemetry source.
7. Publish only deliberately reviewed non-secret fingerprints, such as the compiled Sysmon rules SHA-256 recorded above.

## Validation Date

The environment, registered-agent, service-state, active-collection, PowerShell-policy, and Sysmon-fingerprint inventory was completed on July 14, 2026 local lab time.
