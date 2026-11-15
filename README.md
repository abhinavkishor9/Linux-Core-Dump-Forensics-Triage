# Linux Core Dump Forensics Triage

## Overview

This lab investigates Linux core dump configuration and crash-handling mechanisms on a Rocky Linux 9.8 server.

The investigation focuses on understanding how Linux captures process crash information, identifying the configured core dump handler, checking for available crash artifacts, and reviewing system journal and Wazuh events for supporting evidence.

The objective is to determine whether any process crash artifacts are available for forensic examination without deliberately crashing applications or modifying system configuration.

## Environment

- Operating System: Rocky Linux 9.8
- Kernel: 5.14.0-687.31.1.el9_8.x86_64
- SIEM: Wazuh 4.13.1
- Core dump handler: systemd-coredump
- Core dump utility: coredumpctl
- Core dump storage directory: `/var/lib/systemd/coredump/`
- Investigation date: 16 November 2026

## Investigation Objectives

- Examine the current shell's core dump size limit.
- Identify the kernel core dump handling configuration.
- Check systemd-coredump availability and socket status.
- Identify available core dump records.
- Inspect the configured core dump storage location.
- Review system journal messages for process crashes.
- Correlate relevant events with Wazuh Discover.
- Distinguish actual process crash evidence from unrelated events.

## Key Findings

### Core Dump Configuration

The current root shell reported:

```text
ulimit -c
unlimited
```

The kernel core dump pattern was:

```text
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h %d %F
```

This indicates that core dump data is passed to systemd-coredump for handling.

### systemd-coredump

The systemd-coredump socket was active and listening.

The `coredumpctl` utility was available at:

```text
/usr/bin/coredumpctl
```

However, executing:

```bash
coredumpctl list
```

returned:

```text
No coredumps found.
```

The directory `/var/lib/systemd/coredump/` was also empty.

### Journal Review

The current boot journal contained messages relating to:

- Kernel crash dump memory reservation.
- Crash recovery kernel arming.
- System startup.

These messages do not establish that an application crashed or generated a process core dump.

### Wazuh Discover

A Windows Registry Key deletion event was observed:

| Field | Value |
|---|---|
| Agent ID | 001 |
| Agent name | DESKTOP-9MMM37V |
| Rule ID | 597 |
| Rule level | 5 |
| Event type | Registry key deletion |
| Event source | Syscheck |
| Registry path | `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\webthreatdefusersvc_255de10f\Security` |

This event originated from the Windows endpoint and was unrelated to Linux core dump activity.

## Investigation Assessment

| Area | Finding |
|---|---|
| Current shell core limit | Unlimited |
| Kernel core pattern | systemd-coredump handler |
| systemd-coredump socket | Active |
| coredumpctl utility | Available |
| Core dump records | None found |
| Core dump storage directory | Empty |
| Application crash confirmed | No |
| Wazuh crash-related evidence | Not established |
| Unrelated Wazuh event | Windows registry key deletion |

## Conclusion

The investigation confirmed that the Rocky Linux server is configured to use systemd-coredump for handling process core dumps.

Although the current root shell has an unlimited core dump size limit and the systemd-coredump socket is active, no recorded core dumps were found.

The journal contained kernel crash-recovery initialization messages, but these should not be interpreted as evidence of an application crash.

A Windows registry deletion event appeared in Wazuh Discover, demonstrating the importance of validating agent identity and event context before associating an alert with an investigation.

No evidence of a process crash, malicious exploitation, or core dump tampering was established during this lab.

## Evidence-Based Approach

- Core dump configuration indicates how crash data is handled.
- `coredumpctl` provides visibility into recorded process crashes.
- Storage inspection helps identify available artifacts.
- Journal messages provide additional system context.
- Wazuh events must be correlated with the correct endpoint and activity.

**Follow the evidence, not the assumption.**
