# Public Incident Report — restic/rclone SFTP known_hosts and Integrity Check Failure

## Summary

A self-hosted backup control plane using `restic` and `rclone` was expanded to additional remote endpoints and a second offsite storage provider using SFTP. During validation, some remote backup runs failed because the SFTP host key validation configuration had only been applied on the orchestrator, not on the remote endpoints where `rclone` was actually executed.

A separate integrity check also failed because encrypted environment configuration was not robustly loaded under systemd. Both issues were fixed and verified with successful repository checks and mini-restore tests.

## Environment

Generalized environment:

- Linux backup orchestrator
- Multiple Linux remote endpoints
- `restic` for snapshots
- `rclone` for remote storage transport
- SFTP-based offsite storage
- systemd timers and one-shot services
- encrypted secret file loaded during service execution

## Problem

The backup run produced errors similar to:

```text
ssh: handshake failed: knownhosts: key is unknown
```

The integrity check produced errors similar to:

```text
repo variable missing
```

## Current System State / Observed Behavior

- The orchestrator had SFTP host-key validation configured.
- Remote endpoints did not all have the same `known_hosts` state.
- The backup script executed `restic`/`rclone` on each endpoint, so endpoint-local configuration was required.
- The integrity script logged secret-loading success even though the expected environment variables were unavailable afterwards.

## Root Cause

1. **Wrong configuration scope**: The SFTP host-key configuration was set on the orchestrator, but the storage client ran on the remote endpoints.
2. **Fragile secret loading**: The integrity script did not reliably fail when encrypted config loading did not populate required variables.

## Fix / Resolution

- Added the SFTP storage host key to all endpoints that run `rclone`.
- Configured the rclone remote to use a specific `known_hosts` file.
- Verified remote storage access from every endpoint.
- Re-ran a clean backup.
- Updated the integrity check to decrypt the secret file into a temporary file and source it explicitly.
- Ensured secret-loading failures are reported as failures instead of being silently ignored.

## Verification

Successful verification included:

```text
backup completed successfully
repository check OK
mini-restore OK
integrity status OK
snapshots visible on both providers
```

## Prevention / Hardening

- Validate provider configuration where the provider client actually runs.
- Treat host-key validation as endpoint-local state for remote execution models.
- Avoid suppressing secret-loading errors in systemd services.
- After adding endpoints, run both backup verification and integrity verification.
- Keep backup checks and restore tests separate from snapshot creation.

## Lessons Learned

A green orchestrator configuration does not guarantee green remote execution. In distributed backup models, each remote endpoint needs the runtime prerequisites for the storage backend.

## Disclaimer

This report is a public, sanitized summary. Hostnames, domains, internal paths, credential material and organization-specific security details have been generalized or removed.
