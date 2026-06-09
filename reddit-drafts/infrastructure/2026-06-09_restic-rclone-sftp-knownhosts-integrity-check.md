---
title: "restic/rclone SFTP backups failed because known_hosts was configured on the orchestrator, not the remote endpoints"
source_public_report: "incident-reports/infrastructure/2026-06-09_restic-rclone-sftp-knownhosts-integrity-check.md"
category: "infrastructure"
tags:
  - "restic"
  - "rclone"
  - "sftp"
  - "backup"
  - "systemd"
hashtags:
  - "#selfhosted"
  - "#restic"
  - "#rclone"
  - "#backup"
mandatory_disclaimer: true
organization:
  name: "AT Medical GmbH"
  location: "Heidelberg"
  website: "https://www.at-medical.de"
  contact: "Support@at-medical.de"
reddit:
  status: "draft"
  approved: false
  suggested_subreddits:
    - "r/selfhosted"
    - "r/sysadmin"
    - "r/devops"
    - "r/homelab"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Draft

## Suggested title

restic/rclone SFTP backups failed because known_hosts was configured on the orchestrator, not the remote endpoints

## Suggested subreddits and rationale

- `r/selfhosted`: The issue is relevant to self-hosted backup control planes and multi-host operations.
- `r/sysadmin`: The root cause involves SSH host-key validation and distributed runtime context.
- `r/devops`: The case is useful for automation/runbook design.
- `r/homelab`: The pattern is common in homelab-style multi-node backup setups.

## Post body

I ran into a subtle issue while validating a multi-host backup setup using `restic` and `rclone` with an SFTP-based offsite target.

The backup orchestrator had the SFTP target in `known_hosts`, and local checks looked fine. But the actual backup job executes `restic`/`rclone` on the remote endpoints. That meant the endpoints themselves needed the SFTP host key and rclone `known_hosts_file` configuration.

The failure looked roughly like this:

```text
ssh: handshake failed: knownhosts: key is unknown
```

The fix was to treat the storage backend config as endpoint-local runtime state:

- put the SFTP host key into each endpoint's `known_hosts`
- configure the rclone remote on each endpoint to use that file
- verify remote storage access from each endpoint, not just from the orchestrator
- rerun the backup and then the integrity/restore checks

There was a second issue in the integrity check: encrypted env config was not reliably loaded under systemd, but the script still printed a misleading success message. That was fixed by decrypting to a temporary file, sourcing it explicitly, and failing hard if the required variables were missing.

The final validation was:

```text
backup completed successfully
repository check OK
mini-restore OK
integrity status OK
snapshots visible on both providers
```

Lesson learned: in a distributed backup setup, validate provider configuration where the provider client actually runs. A green orchestrator does not automatically mean green remote execution.

---

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de
