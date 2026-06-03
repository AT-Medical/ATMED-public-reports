# Public Incident Report: Authentik Admin Bootstrap Recovery Pattern

## Summary

During a self-hosted Authentik deployment, administrative access had to be finalized after older assumptions about the default admin account and CLI workflow no longer matched the deployed Authentik version. The issue was resolved by using Authentik shell and the supported admin-group workflow, then verifying login, MFA, service health, and documentation.

This public report is sanitized. It omits organization-specific hostnames, IP addresses, paths, secrets, and internal architecture details.

## Environment

- Self-hosted Authentik deployment
- Docker / Docker Compose
- Reverse proxy in front of Authentik
- PostgreSQL backend
- Redis backend
- MFA enabled for administrative access

## Problem

The expected legacy/default administrative user did not exist. Attempts to use older `createsuperuser` workflows failed because the deployed Authentik version did not support the attempted command pattern and explicitly pointed toward the admin-group workflow.

Relevant sanitized error signatures:

```text
CommandError: user 'akadmin' does not exist
manage.py createsuperuser: error: unrecognized arguments: --username --email
RuntimeError: ak createsuperuser should not be used. Instead, use ak create_admin_group
Environment variable not found, using fallback: POSTGRES_PASSWORD
```

## Current System State / Observed Behavior

- The Authentik service itself was able to start.
- Public login endpoint redirected to the configured authentication flow.
- Administrative account bootstrapping was the failing part.
- A compatibility warning appeared for a PostgreSQL-related environment variable.

## Root Cause

The recovery process used outdated assumptions:

1. The legacy default account was not present.
2. The attempted `createsuperuser` command path was not suitable for the deployed Authentik version.
3. The correct recovery path was to create or update the desired admin user and assign admin permissions using Authentik's supported admin group workflow.

A secondary configuration warning was caused by an expected compatibility environment variable not being present, even though the application-specific database configuration existed.

## Fix / Resolution

The general remediation pattern was:

1. Use Authentik shell to create or update the intended admin user.
2. Add the user to the Authentik admin group using the supported admin-group command.
3. Reset the admin password through the Authentik CLI.
4. Confirm MFA remains active.
5. Address any database environment compatibility warning.
6. Recreate affected containers.
7. Verify container health and public endpoint behavior.
8. Create checksum-protected runbooks for future recovery.

## Verification

Verification should include:

- Authentik server healthy.
- Worker healthy.
- PostgreSQL healthy.
- Redis healthy.
- Public endpoint redirects to the expected authentication flow.
- Admin login succeeds.
- MFA remains active.
- Logs show no fresh errors or warnings after remediation.
- Recovery runbooks are written and checksum-verified.

## Prevention / Hardening

- Avoid relying on default admin-user assumptions.
- Keep version-aware Authentik recovery notes.
- Add WebAuthn/passkeys as an additional admin factor.
- Keep secrets out of runbooks and repositories.
- Store recovery material in an offline break-glass vault.
- Rotate exposed or temporarily backed-up secrets.
- Add audit notifications for privileged identity events.

## Lessons Learned

Identity-provider recovery procedures must be tested after version changes. A service can be healthy while the administrative recovery path is still incomplete. The recovery path should be documented, verified, and protected with checksums just like any other critical infrastructure runbook.

## Disclaimer

This is a sanitized technical report based on a real troubleshooting and remediation process. It intentionally excludes organization-specific details, security-sensitive paths, hostnames, IP addresses, and secrets.
