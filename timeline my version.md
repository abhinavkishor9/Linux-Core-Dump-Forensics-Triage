# Investigation Timeline 

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

