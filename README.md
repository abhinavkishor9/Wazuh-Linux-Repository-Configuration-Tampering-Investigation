# Lab 134 – Linux Repository Configuration Tampering Investigation

## Overview

This lab focuses on investigating Linux package repository configuration changes using Auditd and Wazuh on Rocky Linux 9.8.

Linux package managers depend on repository configuration files to identify software sources, retrieve package metadata, and download packages. Unauthorized modifications to these configurations could redirect package downloads, disable trusted repositories, or weaken package verification settings.

The investigation demonstrates how a SOC analyst can monitor repository configuration files, identify changes, examine audit records, and validate the availability of security telemetry in Wazuh.

## Objectives

- Understand Linux repository configuration and its security implications.
- Identify repository configuration files on Rocky Linux.
- Establish a baseline of repository settings.
- Monitor repository configuration changes using Auditd.
- Perform a controlled modification of a dedicated test repository file.
- Investigate file modification activity through Auditd records.
- Examine the availability and classification of audit telemetry in Wazuh.
- Restore the test configuration and clean up temporary monitoring rules.

## Environment

| Component | Details |
|---|---|
| Operating System | Rocky Linux 9.8 |
| Package Manager | DNF 4.14.0 |
| Audit Framework | Auditd 3.1.5 |
| SIEM | Wazuh 4.13.1 |
| Repository Configuration Directory | `/etc/yum.repos.d/` |
| Global DNF Configuration | `/etc/dnf/dnf.conf` |
| Audit Log | `/var/log/audit/audit.log` |

## Investigation Scope

The primary monitoring target is:

`/etc/yum.repos.d/`

A dedicated test repository file is used:

`/etc/yum.repos.d/lab134-test.repo`

The test repository configuration was designed to contain:

```ini
[lab134-test]
name=Lab 134 Test Repository
baseurl=https://example.invalid/repository/
enabled=0
gpgcheck=1
```

The controlled test involves changing the `gpgcheck` setting from `1` to `0`, then examining the resulting file and audit evidence.

The repository remains disabled. The reserved `.invalid` domain is used as a placeholder rather than a real package source.

## Monitoring Approach

Auditd directory monitoring:

```bash
sudo auditctl -w /etc/yum.repos.d/ -p wa -k lab134_repo_changes
```

The rule monitors:

- Write operations.
- File attribute changes.

The audit key `lab134_repo_changes` provides a way to search for related records.

## Investigation Workflow

1. Identify existing repository configuration files.
2. Inspect repository definitions and global DNF settings.
3. Record file metadata and SHA-256 checksums.
4. Verify Auditd status.
5. Configure a targeted Auditd watch.
6. Create a dedicated test repository file.
7. Modify the test repository's GPG verification setting.
8. Compare file checksums and metadata.
9. Examine Auditd records using the lab-specific key.
10. Review relevant telemetry in Wazuh.
11. Restore the original test configuration.
12. Remove the test file and temporary Auditd rule.

## Evidence and Findings

The lab is designed to establish whether a controlled repository configuration modification can be observed through Linux audit telemetry.

The following evidence should be retained:

- Initial repository configuration.
- Original file metadata and checksum.
- Modified test repository configuration.
- Modified file metadata and checksum.
- Auditd records associated with the test activity.
- Relevant Wazuh events, if received.
- Cleanup verification.

**Evidence limitation:** Auditd and Wazuh findings should be documented only after reviewing the actual command output and event records. A configuration-change event must not automatically be classified as malicious activity.

## Security Relevance

Repository configuration tampering can affect the software supply chain by changing where packages are retrieved or weakening package verification settings.

Monitoring repository configuration files provides visibility into changes that may require further investigation.

A SOC analyst should correlate the modified file, process information, user identity, timestamps, and subsequent package-management activity before determining whether the change was unauthorized.

## Conclusion

This investigation demonstrates an evidence-based approach to monitoring Linux repository configuration changes using Auditd and Wazuh.

The key objective is to distinguish a confirmed configuration modification from an assumption of malicious activity and to establish whether the available telemetry supports the investigation.
