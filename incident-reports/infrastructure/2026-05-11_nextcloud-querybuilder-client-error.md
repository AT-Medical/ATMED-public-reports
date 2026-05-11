# Nextcloud Desktop Client Failure After Login-v2 Authentication

## Summary

A Nextcloud desktop client completed the browser-based OAuth/Login-v2 flow successfully but failed immediately afterward with a server-side exception.

## Environment

- Nextcloud 33
- Docker deployment
- Reverse proxy setup

## Observed Behavior

The server itself remained reachable and `/status.php` returned a healthy installation state.

After successful browser authorization, the desktop client failed with:

```text
Call to undefined method OCP\\DB\\QueryBuilder\\QueryBuilder::execute()
```

## Root Cause

The issue was most likely caused by an incompatible or outdated Nextcloud app still using a deprecated QueryBuilder method after upgrading to Nextcloud 33.

## Investigation Steps

```bash
php occ app:list
php occ app:update --all
php occ log:watch
```

Additional checks:

- reverse proxy TLS configuration
- accidental mTLS/client-certificate enforcement
- SSO or OAuth-related apps

## Resolution Strategy

1. Identify incompatible apps from logs.
2. Update all apps.
3. Disable problematic app if necessary.
4. Retest desktop client login.

## Prevention

- Perform compatibility testing before major upgrades.
- Maintain a staging environment.
- Separate admin mTLS policies from user-facing applications.

## Lessons Learned

TLS/client-certificate dialogs in desktop clients can sometimes be misleading side effects of backend application failures rather than true TLS configuration problems.

## Disclaimer

This report has been sanitized and generalized. Infrastructure-specific identifiers, domains, paths, and security details were intentionally removed.
