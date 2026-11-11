# Lab 134 – Troubleshooting Notes

## 1. Editing Repository Files Using vi

The test repository file is created or edited using:

```bash
sudo vi /etc/yum.repos.d/lab134-test.repo
```

To save and exit:

1. Press `Esc`.
2. Type `:wq`.
3. Press `Enter`.

Useful vi commands:

| Command | Action |
|---|---|
| `:wq` | Save and exit. |
| `:w` | Save without exiting. |
| `:q!` | Exit without saving. |
| `:x` | Save changes and exit. |

## 2. Repository Configuration Safety

Existing Rocky Linux repository files should not be modified for this exercise.

Use a dedicated test file:

`/etc/yum.repos.d/lab134-test.repo`

Keep the test repository disabled:

```ini
enabled=0
```

The placeholder URL:

```ini
baseurl=https://example.invalid/repository/
```

is not intended to be used for package downloads.

## 3. Auditd Watch Verification

After adding the directory watch:

```bash
sudo auditctl -w /etc/yum.repos.d/ -p wa -k lab134_repo_changes
```

Verify the loaded rule:

```bash
sudo auditctl -l | grep lab134_repo_changes
```

If no output appears, the expected rule is not visible in the loaded rules.

Check Auditd status before troubleshooting event collection:

```bash
sudo auditctl -s
```

Review the `enabled` and `lost` values.

## 4. Searching Auditd Records

Use:

```bash
sudo ausearch -k lab134_repo_changes -i
```

If no records are returned, do not immediately conclude that the file was not modified.

Check:

- Whether the watch was active before the modification.
- Whether Auditd was running.
- Whether the modification targeted the watched directory.
- Whether the audit key was specified correctly.
- Whether audit events were lost.

## 5. Understanding Multiple Auditd Records

A single audited operation can generate multiple records.

For example:

- `SYSCALL`
- `PATH`
- `CWD`
- `PROCTITLE`

These records may share an audit event identifier.

They should be correlated as parts of the same event rather than counted as separate file modifications.

## 6. Understanding Wazuh Events

A configuration-change event generated when an Auditd rule is added is not evidence that a repository file was modified.

Similarly, receiving an Auditd event in Wazuh does not automatically mean that a dedicated repository-tampering alert exists.

Check the raw event and decoded fields before drawing conclusions.

## 7. Checksum Interpretation

A different SHA-256 checksum before and after modification indicates that the file content changed.

It does not independently establish:

- Who modified the file.
- Why it was modified.
- Whether the modification was authorized.
- Whether the repository was subsequently used.

Correlate checksum evidence with Auditd records and timestamps.

## 8. Safe Cleanup

Remove only the test repository file:

```bash
sudo rm -f /etc/yum.repos.d/lab134-test.repo
```

Remove only the Lab 134 audit watch:

```bash
sudo auditctl -W /etc/yum.repos.d/ -p wa -k lab134_repo_changes
```

Verify the result:

```bash
sudo auditctl -l | grep lab134_repo_changes
```

Avoid deleting unrelated Auditd rules because they may belong to earlier investigations.
