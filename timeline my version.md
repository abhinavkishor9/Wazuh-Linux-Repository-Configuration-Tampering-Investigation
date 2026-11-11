# Investigation Timeline

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

