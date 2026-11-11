# Investigation Notes

## 1. Investigation Objective

The objective is to investigate modifications to Linux package repository configuration files and determine whether the activity can be identified through Auditd and Wazuh.

Repository configuration changes can affect package sources, repository availability, and package verification settings.

## 2. Repository Configuration Discovery

The investigation begins by examining:

- `/etc/yum.repos.d/`
- `/etc/dnf/dnf.conf`
- Existing `.repo` files.

The repository files provide information about configured package sources and their security-related settings.

The following configuration fields are particularly relevant:

| Field | Investigation relevance |
|---|---|
| `baseurl` | Identifies the configured package source. |
| `mirrorlist` | Identifies a repository mirror list. |
| `metalink` | Identifies repository metadata discovery settings. |
| `enabled` | Indicates whether a repository is enabled. |
| `gpgcheck` | Controls package signature verification. |

## 3. Baseline Collection

Before performing a controlled modification, collect:

- Repository file contents.
- File ownership and permissions.
- File modification timestamps.
- SHA-256 checksums.

Example:

```bash
sudo stat /etc/yum.repos.d/*.repo
sudo sha256sum /etc/yum.repos.d/*.repo
```

These values provide a baseline for subsequent comparison.

## 4. Auditd Monitoring

A temporary directory watch is configured:

```bash
sudo auditctl -w /etc/yum.repos.d/ -p wa -k lab134_repo_changes
```

The watch monitors write and attribute-change activity under the repository configuration directory.

The audit key allows related records to be searched:

```bash
sudo ausearch -k lab134_repo_changes -i
```

## 5. Controlled Repository Configuration Change

A dedicated test file is used instead of modifying an existing Rocky Linux repository.

Test file:

`/etc/yum.repos.d/lab134-test.repo`

Intended initial configuration:

```ini
[lab134-test]
name=Lab 134 Test Repository
baseurl=https://example.invalid/repository/
enabled=0
gpgcheck=1
```

The controlled modification changes:

```ini
gpgcheck=1
```

to:

```ini
gpgcheck=0
```

The repository remains disabled throughout the test.

This simulates a security-relevant configuration modification without changing the active repository definitions.

## 6. File Integrity Comparison

The SHA-256 checksum is collected before and after the modification.

A difference between the checksums establishes that the file content changed.

File metadata can also be examined using:

```bash
sudo stat /etc/yum.repos.d/lab134-test.repo
```

A changed modification timestamp provides additional context.

A checksum difference alone does not identify the responsible process or establish malicious intent.

## 7. Auditd Evidence Examination

Search for the lab-specific audit key:

```bash
sudo ausearch -k lab134_repo_changes -i
```

Relevant audit information includes:

- `SYSCALL` – system call details.
- `PATH` – affected file path.
- `PROCTITLE` – process command-line information, when available.
- `CWD` – current working directory.
- `exe` – executable path.
- `comm` – process name.
- `uid` – process user ID.
- `auid` – login user identity, when available.
- `success` – operation result.

Multiple records may belong to one Auditd event. They should be correlated using the audit event identifier rather than interpreted independently.

## 8. Wazuh Telemetry Examination

Wazuh Discover can be used to search for relevant audit events.

Example search:

```text
full_log: "lab134-test.repo"
```

The following fields should be examined when available:

- `@timestamp`
- `agent.id`
- `agent.name`
- `data.audit.type`
- `data.audit.exe`
- `data.audit.uid`
- `data.audit.auid`
- `rule.id`
- `rule.description`
- `rule.level`
- `full_log`

The presence of an Auditd event in Wazuh establishes telemetry collection. A dedicated alert is a separate outcome and should be verified independently.

## 9. Evidence Assessment

The investigation should answer:

1. Which repository file was changed?
2. Which configuration value was modified?
3. Did the file checksum change?
4. Which executable performed the operation?
5. Which user identity was associated with the event?
6. Was the operation successful?
7. Did Wazuh receive the relevant audit event?
8. Was a corresponding Wazuh alert generated?
9. Was any subsequent package-management activity identified?

## 10. Investigation Limitations

The following distinctions are important:

- A checksum difference confirms content modification, not malicious intent.
- A file modification timestamp does not independently identify the process.
- An Auditd configuration-change event is not the same as a repository file modification.
- A repository modification does not establish that packages were downloaded or installed.
- A Wazuh event does not necessarily mean a dedicated detection rule generated an alert.

Only validated evidence should be used in the final investigation conclusion.

## 11. Cleanup

Restore the test repository configuration to its original value:

```ini
gpgcheck=1
```

Remove the dedicated test file:

```bash
sudo rm -f /etc/yum.repos.d/lab134-test.repo
```

Remove only the Lab 134 Auditd watch:

```bash
sudo auditctl -W /etc/yum.repos.d/ -p wa -k lab134_repo_changes
```

Verify that the test file and lab-specific rule are no longer present.

Do not use `auditctl -D`, because it removes all loaded Auditd rules.

