# Investigation Timeline – Linux Core Dump Forensics Triage

**Date:** 16 November 2026  
**System:** Rocky Linux 9.8  
**SIEM:** Wazuh 4.13.1

| Sequence | Activity | Evidence / Observation |
|---|---|---|
| 1 | Started Wazuh services | Start commands executed for manager, indexer, and dashboard |
| 2 | Identified server IP | `192.168.174.128` |
| 3 | Checked core dump size limit | `unlimited` for the current root shell |
| 4 | Examined kernel core pattern | Core dump handling configured through systemd-coredump |
| 5 | Checked systemd-coredump socket | Active and listening |
| 6 | Verified coredumpctl availability | `/usr/bin/coredumpctl` |
| 7 | Listed recorded core dumps | No coredumps found |
| 8 | Inspected core dump storage | `/var/lib/systemd/coredump/` was empty |
| 9 | Reviewed current boot journal | Kernel crash-recovery initialization messages observed |
| 10 | Reviewed Wazuh Discover | Windows registry deletion event from agent 001 |
| 11 | Assessed collected evidence | No confirmed application crash or core dump artifact |

## Key Observations

- The system uses systemd-coredump as its configured core dump handler.
- The current root shell has an unlimited core dump size limit.
- The systemd-coredump socket was active.
- No recorded core dumps were found.
- The standard systemd-coredump storage directory was empty.
- Journal messages related to crash recovery did not establish an application crash.
- The Wazuh event originated from a Windows endpoint and was unrelated to this investigation.

## Final Assessment

The investigation confirmed the core dump handling configuration but did not identify any available process core dump artifacts.

No confirmed application crash, exploitation, or core dump tampering was established from the collected evidence.
