# Investigation Notes – Linux Core Dump Forensics Triage

## 1. Investigation Summary

The objective was to examine the Linux core dump configuration, determine whether the server had recorded any process crashes, and identify available forensic artifacts.

The investigation was conducted on Rocky Linux 9.8 using systemd-coredump, coredumpctl, the system journal, and Wazuh Discover.

No application was intentionally crashed, and no core dump configuration was modified.

## 2. Wazuh Service Initialization

The following commands were executed:

```bash
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-dashboard
```

The server IP address was identified using:

```bash
hostname -I
```

Observed output:

```text
192.168.174.128
```

The service start commands were issued before beginning the investigation.

## 3. Core Dump Size Limit

Command:

```bash
ulimit -c
```

Observed output:

```text
unlimited
```

Finding:

The current root shell has no shell-imposed core dump size limit.

This value applies to the current shell and processes inheriting its resource limits. It does not independently prove that every process on the server can generate a core dump.

## 4. Kernel Core Dump Configuration

Command:

```bash
cat /proc/sys/kernel/core_pattern
```

Observed output:

```text
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h %d %F
```

Finding:

The kernel is configured to pipe core dump data to the systemd-coredump handler.

The pattern contains metadata placeholders for information such as process ID, user ID, group ID, signal number, timestamp, and other process details.

This confirms the configured handling mechanism, not the existence of an actual crash artifact.

## 5. systemd-coredump Socket

Command:

```bash
systemctl status systemd-coredump.socket --no-pager
```

Observed status:

```text
systemd-coredump.socket - Process Core Dump Socket
Loaded: loaded
Active: active (listening)
Listen: /run/systemd/coredump
Accepted: 0
Connected: 0
```

Finding:

The systemd-coredump socket was active and listening.

The socket provides an interface for receiving core dump data. Its active state does not mean that a process has crashed.

## 6. coredumpctl Availability

Command:

```bash
command -v coredumpctl
```

Observed output:

```text
/usr/bin/coredumpctl
```

Finding:

The coredumpctl utility is available for inspecting systemd-managed core dump records.

## 7. Available Core Dump Records

Command:

```bash
coredumpctl list
```

Observed output:

```text
No coredumps found.
```

Finding:

No recorded core dumps were available through coredumpctl at the time of investigation.

This does not prove that no process has ever crashed. Core dump generation and retention depend on system configuration and other conditions.

## 8. Core Dump Storage Inspection

Command:

```bash
ls -lah /var/lib/systemd/coredump/
```

Observed output:

```text
total 0
drwxr-xr-x. 2 root root  6 Aug  1 16:35 .
drwxr-xr-x. 7 root root 98 Jun 24 05:39 ..
```

Finding:

The standard systemd-coredump storage directory existed but contained no core dump files.

This observation is consistent with the empty coredumpctl listing.

## 9. System Journal Review

Command:

```bash
journalctl -b --no-pager | grep -Ei 'segfault|core dumped|dumped core|crash'
```

Relevant output included:

```text
Nov 16 04:04:07 localhost kernel:
crashkernel reserved: 0x00000000af000000 - 0x00000000bf000000 (256 MB)

Nov 16 04:04:44 localhost.localdomain systemd[1]:
Starting Crash recovery kernel arming...

Nov 16 04:04:50 localhost.localdomain systemd[1]:
Finished Crash recovery kernel arming.
```

Finding:

The journal recorded kernel crash-recovery initialization and memory reservation.

These messages relate to kernel crash recovery preparation. They do not establish an application segmentation fault or a process core dump.

No confirmed application crash message was identified in the displayed results.

## 10. Wazuh Discover Review

The Wazuh Discover search returned a Windows registry deletion event.

Event details:

| Field | Value |
|---|---|
| Agent ID | 001 |
| Agent name | DESKTOP-9MMM37V |
| Rule ID | 597 |
| Rule level | 5 |
| Rule description | Registry Key Entry Deleted. |
| Event source | Syscheck |
| Event type | Registry key deleted |
| Mode | Scheduled |

Registry path:

```text
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\webthreatdefusersvc_255de10f\Security
```

Analysis:

The event originated from the Windows endpoint `DESKTOP-9MMM37V`, not the Rocky Linux server.

It records a registry key deletion detected by Wazuh Syscheck. It does not provide evidence of a Linux process crash or core dump operation.

The event should therefore be excluded from the Linux core dump findings.

## 11. Investigation Assessment

| Evidence area | Result |
|---|---|
| Current shell core limit | Unlimited |
| Kernel core pattern | systemd-coredump |
| systemd-coredump socket | Active |
| coredumpctl | Available |
| Recorded core dumps | None found |
| Core dump storage | Empty |
| Application crash evidence | Not established |
| Kernel crash-recovery initialization | Observed |
| Relevant Wazuh crash event | Not established |
| Unrelated Windows event | Observed |

## 12. Conclusion

The Rocky Linux server is configured to use systemd-coredump for handling process core dumps.

The current root shell has an unlimited core dump size limit, and the systemd-coredump socket is active. However, coredumpctl returned no records, and the standard storage directory was empty.

The journal messages related to crash-recovery initialization should not be mistaken for evidence of an application crash.

The Wazuh event observed during the investigation was a Windows registry deletion event from another endpoint and was not relevant to this Linux investigation.

No confirmed process crash or malicious activity was established from the collected evidence.
