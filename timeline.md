# Lab 134 – Investigation Timeline

## Environment

- Operating System: Rocky Linux 9.8
- Package Manager: DNF 4.14.0
- Audit Framework: Auditd 3.1.5
- SIEM: Wazuh 4.13.1
- Monitored Directory: `/etc/yum.repos.d/`

## Investigation Timeline

| Stage | Activity | Evidence / Purpose |
|---|---|---|
| 01 | Repository discovery | Identify existing repository configuration files. |
| 02 | Configuration inspection | Review repository URLs, enabled status, and GPG verification settings. |
| 03 | Baseline collection | Record file metadata and SHA-256 checksums. |
| 04 | Auditd verification | Check audit service and audit subsystem status. |
| 05 | Audit rule creation | Configure a watch for repository directory changes. |
| 06 | Test repository preparation | Create `lab134-test.repo` with a disabled test repository. |
| 07 | Controlled modification | Change the test file's `gpgcheck` value from `1` to `0`. |
| 08 | Integrity comparison | Compare file checksums and metadata. |
| 09 | Audit investigation | Search for records using `lab134_repo_changes`. |
| 10 | SIEM validation | Examine relevant events in Wazuh Discover. |
| 11 | Restoration | Return the test configuration to its original setting. |
| 12 | Cleanup | Remove the test file and temporary Auditd watch. |

## Evidence Status

The timeline above describes the investigation sequence. Actual event timestamps, audit serial numbers, process details, Wazuh rule IDs, and alert classifications must be added from the collected terminal and dashboard evidence.

Do not assign timestamps or event identifiers without verified records.

## Final Assessment

The investigation is intended to establish whether the controlled repository configuration modification was recorded by Auditd and whether the resulting telemetry was available in Wazuh.

The conclusion must distinguish confirmed configuration changes from unverified malicious intent or subsequent package installation activity.
