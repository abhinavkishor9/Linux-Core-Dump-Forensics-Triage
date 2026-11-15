# Linux-Core-Dump-Forensics-Triage
## Overview
### 1. What Is a Core Dump?

A core dump is a snapshot of a process's memory and execution state captured when a process terminates abnormally.

It can contain information about the process at the time of termination, including:

- Process memory.
- CPU register state.
- Stack information.
- Thread information.
- Loaded libraries.
- Execution context.

Core dumps are useful for debugging application crashes and investigating unexpected process termination.

### 2. Why Are Core Dumps Important in DFIR?

Core dumps can provide valuable evidence during security investigations.

They may help analysts:

- Identify which process terminated unexpectedly.
- Understand the circumstances of a process crash.
- Examine memory-related failures.
- Investigate possible memory corruption.
- Identify suspicious application behavior.
- Reconstruct portions of a process's execution state.

However, a process crash does not automatically indicate malicious activity.

Application bugs, invalid memory access, and resource-related problems can also cause crashes.

### 3. Common Causes of Process Crashes

#### Segmentation Fault

A segmentation fault occurs when a process attempts to access memory in an invalid or unauthorized way.

#### Memory Corruption

Unexpected modification of memory can cause application instability or abnormal termination.

#### Programming Errors

Software defects such as invalid pointers and incorrect memory handling can cause crashes.

#### Resource-Related Problems

Resource exhaustion and other system limitations may contribute to unexpected process termination.

### 4. Linux Core Dump Configuration

Linux provides several mechanisms for controlling core dump generation and handling.

#### ulimit

The command:

```bash
ulimit -c
```

Displays the core dump size limit for the current shell.

An output of `0` indicates that the current shell imposes a zero core dump size limit. An output of `unlimited` means no shell-imposed maximum.

#### kernel.core_pattern

The configuration can be examined using:

```bash
cat /proc/sys/kernel/core_pattern
```

This determines how the kernel names core dump files or passes core dump data to a handler.

A pattern beginning with `|` indicates that core dump data is piped to a configured program.

### 5. systemd-coredump

systemd-coredump is a systemd component that can receive and manage process core dumps.

It can collect crash information and make recorded crash metadata available for investigation.

The socket can be checked using:

```bash
systemctl status systemd-coredump.socket
```

Its availability does not prove that any process has crashed.

### 6. coredumpctl

`coredumpctl` is a utility for examining core dump records managed by systemd-coredump.

Useful commands include:

```bash
coredumpctl list
```

Lists available core dump records.

```bash
coredumpctl info
```

Displays information about recorded core dumps.

```bash
coredumpctl info <PID>
```

Displays information associated with a particular process ID, when a matching record exists.

### 7. Core Dump Storage

Systemd-coredump may store core dump files under:

```text
/var/lib/systemd/coredump/
```

The actual storage behavior depends on the system's configuration and retention settings.

Core dumps should be treated as sensitive forensic artifacts because process memory may contain:

- Passwords.
- Authentication tokens.
- API credentials.
- Encryption keys.
- Personal or confidential information.

Access to these files should be restricted and evidence should be preserved appropriately.

### 8. Core Dumps and Wazuh

Wazuh can provide supporting visibility into process-related and system activity through collected logs, audit events, and configured monitoring.

However, Wazuh does not automatically provide a dedicated alert for every process crash or core dump.

Analysts should correlate:

- Process crash messages.
- System journal entries.
- Core dump metadata.
- File monitoring events.
- Relevant endpoint security events.

A missing alert does not prove that a process did not crash.

### 9. Application Core Dumps vs. Kernel Crash Dumps

These are two different mechanisms.

| Application core dump | Kernel crash dump |
|---|---|
| Captures an individual process's state | Captures information related to a kernel crash |
| Used for application debugging and forensics | Used for kernel crash analysis |
| May be managed by systemd-coredump | May use kdump and reserved crash memory |
| Can be examined using coredumpctl and GDB | Commonly analyzed using kernel crash tools |

Kernel messages about crashkernel memory reservation do not establish that an application generated a core dump.

### 10. Investigation Approach

A practical Linux core dump investigation should include:

1. Checking the current shell's core dump limit.
2. Examining the kernel core pattern.
3. Checking the systemd-coredump handling mechanism.
4. Identifying available core dump records.
5. Inspecting the configured storage location.
6. Reviewing journal messages for process crashes.
7. Correlating relevant events with Wazuh.
8. Assessing the evidence without assuming malicious activity.

### Key Takeaway

Core dumps can provide valuable information about process execution and memory state during an abnormal termination.

However, configuration alone does not establish that a crash occurred. Analysts must identify actual artifacts and correlate them with system and security events.

**Follow the evidence, not the assumption.**

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

## Lab Objectives

The objective of this lab is to develop practical skills in identifying and assessing Linux process crash artifacts.

By completing this lab, the analyst should be able to:

- Explain the purpose of core dumps in Linux systems.
- Identify common causes of abnormal process termination.
- Examine process core dump limits and kernel handling configuration.
- Understand the role of systemd-coredump and coredumpctl.
- Determine whether recorded core dump artifacts are available.
- Examine core dump storage and relevant system journal messages.
- Differentiate application core dumps from kernel crash dumps.
- Understand the potential exposure of sensitive information within process memory.
- Correlate relevant system activity with Wazuh monitoring events.
- Distinguish confirmed crash evidence from unrelated events and assumptions.
- Apply evidence-preservation practices when handling potentially sensitive crash artifacts.

The final objective is to produce an evidence-based assessment of the server's core dump configuration and available forensic artifacts.

## Lab Scenario

### Scenario

You are working as a SOC analyst responsible for monitoring a Rocky Linux server that hosts enterprise services and security monitoring components.

During routine monitoring, an unexpected process termination is reported. The incident requires further examination to understand whether the termination was caused by a normal application failure, a memory-related issue, or potentially suspicious process behavior.

You are assigned to investigate the server's core dump handling configuration and determine whether any crash artifacts are available for forensic analysis.

### Investigation Requirements

Your task is to:

- Examine the current shell's core dump size limit.
- Review the kernel's core dump handling configuration.
- Check whether systemd-coredump is available and active.
- Identify recorded process crash information.
- Inspect the configured core dump storage location.
- Review system journal messages for crash-related activity.
- Examine relevant Wazuh events for supporting evidence.
- Determine whether the available evidence confirms an application crash.

### Investigation Considerations

A core dump can contain a process's memory and execution state, making it valuable during forensic analysis.

However, the presence of a core dump does not automatically indicate malicious activity. Legitimate software defects, memory access violations, and resource-related failures can also cause process termination.

The absence of a core dump does not prove that no process crashed. Generation may be restricted, storage may be unavailable, or previous artifacts may have been removed.

Kernel crash-recovery initialization must also be distinguished from an application-level process crash.

Core dump files may contain sensitive information and should be handled carefully.

### Expected Outcome

At the end of the investigation, you should be able to explain the server's core dump handling mechanism, determine whether recorded crash artifacts are available, and identify what additional evidence would be required for deeper analysis.

The final report should distinguish configuration evidence, actual crash artifacts, supporting journal events, and unrelated SIEM alerts.

No process should be deliberately crashed merely to generate evidence on the existing server.

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

