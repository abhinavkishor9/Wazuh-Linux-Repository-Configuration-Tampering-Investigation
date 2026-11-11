# Wazuh-Linux-Repository-Configuration-Tampering-Investigation
## Overview
### Introduction

Linux package managers rely on repository configurations to identify trusted sources from which software packages and metadata can be retrieved.

On Rocky Linux, DNF uses repository definitions stored primarily under `/etc/yum.repos.d/`. These definitions determine repository locations, availability, and package verification settings.

Changes to these configurations can affect the integrity of the software supply chain.

### Repository Configuration

A repository configuration file generally contains:

- Repository identifier.
- Repository name.
- Package source URL.
- Repository enabled or disabled status.
- GPG signature verification settings.

Example:

```ini
[example-repository]
name=Example Repository
baseurl=https://example.com/repository/
enabled=1
gpgcheck=1
```

The `baseurl` setting identifies the package source, while `enabled` determines whether the repository is available to DNF.

The `gpgcheck` setting controls package signature verification.

### Repository Configuration Tampering

Repository configuration tampering occurs when repository settings are changed without appropriate authorization.

Potentially suspicious modifications include:

- Replacing a trusted package source with an unfamiliar URL.
- Adding an unexpected repository.
- Disabling a legitimate repository.
- Disabling package signature verification.
- Modifying repository settings to influence future package downloads.

Such changes may expose the system to untrusted software sources.

However, configuration changes can also result from legitimate administrative maintenance. Context is necessary to determine whether the activity is suspicious.

### Role of Auditd

Auditd records security-relevant activity on Linux systems.

A directory watch can monitor repository configuration changes:

```bash
sudo auditctl -w /etc/yum.repos.d/ -p wa -k lab134_repo_changes
```

The rule monitors write and attribute-change operations.

Audit records can provide information about the affected file, executable, user identity, system call, and operation result.

### Role of Wazuh

Wazuh can collect and analyze Auditd events from the Linux system.

It provides a central location to examine relevant telemetry, correlate activity, and evaluate whether existing detection rules generate alerts.

A received audit event and a generated security alert are different outcomes.

### Investigation Methodology

A repository configuration investigation should follow a structured process:

1. Identify existing repository definitions.
2. Preserve the original configuration.
3. Record file metadata and checksums.
4. Monitor repository configuration changes.
5. Identify the affected file and modified settings.
6. Examine Auditd process and user information.
7. Correlate the event with Wazuh telemetry.
8. Review subsequent package-management activity when relevant.
9. Restore the configuration and validate cleanup.

### Evidence Interpretation

Different evidence sources answer different questions.

- File checksums establish whether file content changed.
- File metadata provides timestamps, ownership, and permissions.
- Auditd records help identify the process and operation.
- Wazuh events establish whether relevant telemetry reached the SIEM.
- DNF transaction records can help determine whether package operations occurred.

No single source necessarily establishes the complete sequence of activity.

### Conclusion

Repository configuration monitoring provides visibility into changes that may affect Linux package sources and software integrity.

An effective investigation combines file integrity evidence, Auditd telemetry, process context, and SIEM correlation.

The objective is to establish what changed, how it changed, and whether the available evidence supports a conclusion of unauthorized activity.


This lab focuses on investigating Linux package repository configuration changes using Auditd and Wazuh on Rocky Linux 9.8.

Linux package managers depend on repository configuration files to identify software sources, retrieve package metadata, and download packages. Unauthorized modifications to these configurations could redirect package downloads, disable trusted repositories, or weaken package verification settings.

The investigation demonstrates how a SOC analyst can monitor repository configuration files, identify changes, examine audit records, and validate the availability of security telemetry in Wazuh.

## Lab Objectives

### Primary Goal

Develop practical skills for investigating changes to Linux package repository configurations and identifying activity that could affect the integrity of software sources.

### Technical Objectives

1. Understand how DNF uses repository definitions to retrieve package metadata and software.
2. Identify the repository configuration files and global package-manager settings on Rocky Linux.
3. Examine repository URLs, enabled status, and GPG verification settings.
4. Establish an initial file-integrity baseline using metadata and SHA-256 checksums.
5. Configure Auditd to monitor repository configuration changes.
6. Perform a controlled modification using a dedicated test repository file.
7. Identify relevant file modification records and associated process information.
8. Examine how Auditd telemetry is represented in Wazuh.
9. Correlate file integrity evidence, audit records, and SIEM events.
10. Restore the original test configuration and remove temporary monitoring artifacts.

### Investigation Skills

- Recognize potentially suspicious repository configuration changes.
- Distinguish file modification evidence from evidence of malicious intent.
- Identify the difference between audit rule changes and monitored file activity.
- Understand the difference between telemetry collection and alert generation.
- Correlate repository changes with subsequent package-management activity.

### Expected Learning Outcome

By completing this lab, the analyst should be able to explain how repository configuration changes can affect Linux software integrity and demonstrate an evidence-based method for investigating such activity using Auditd and Wazuh.


## Lab Scenario

### Scenario

A security monitoring team is reviewing activity on a Rocky Linux server after identifying a potential change to its package repository configuration.

Repository settings determine where the system retrieves software packages and whether package signatures are verified. An unexpected modification could redirect package downloads to an unfamiliar source or weaken an existing security control.

The SOC analyst is assigned to examine the repository configuration and determine what changed, which process performed the operation, and whether the activity was recorded by the system's auditing framework.

### Investigation Context

The analyst begins by identifying the existing repository configuration files and recording their original settings.

To avoid disrupting active package sources, the investigation uses a separate test repository file. A controlled modification is made to its GPG verification setting while the file remains associated with a disabled test repository.

Auditd monitors the repository configuration directory so that the modification can be examined through the resulting audit records.

### Analyst Tasks

The analyst must:

- Identify the repository configuration files present on the server.
- Establish a baseline of repository settings and file integrity.
- Monitor repository configuration changes using Auditd.
- Examine the controlled modification and identify the affected file.
- Investigate the associated process and user information.
- Determine whether the relevant telemetry reaches Wazuh.
- Establish whether any subsequent package-management activity is supported by evidence.
- Restore the original test configuration and remove temporary monitoring rules.

### Investigation Challenge

The analyst must avoid treating every repository configuration change as malicious.

A file modification may be legitimate administrative activity. An Auditd rule-creation event does not prove that a repository file changed, and a file modification does not establish that a package was downloaded or installed.

The investigation must correlate timestamps, file integrity information, process details, Auditd records, and Wazuh telemetry before reaching a conclusion.

### Expected Outcome

The analyst should produce an evidence-based investigation documenting the configuration change, the available audit evidence, the SIEM validation results, and any limitations.

The final assessment should clearly distinguish confirmed activity from assumptions about unauthorized access or malicious intent.

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

