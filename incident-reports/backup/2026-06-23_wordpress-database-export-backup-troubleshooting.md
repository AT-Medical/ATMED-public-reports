---
title: "WordPress Database Export Failed in Docker Backup Workflow"
date: "2026-06-23"
category: "backup"
tags:
  - "wordpress"
  - "docker"
  - "mariadb"
  - "backup"
  - "restic"
source_internal_report: "internal-private-reference-only"
sanitation_status: "draft"
approval_status: "not-approved"
public_release: false
reddit:
  draft_available: true
  approved: false
  suggested_subreddits:
    - "r/selfhosted"
    - "r/docker"
    - "r/sysadmin"
    - "r/homelab"
  selected_subreddit: null
  posted_url: null
---

# WordPress Database Export Failed in Docker Backup Workflow

## Summary

A WordPress backup workflow initially produced a Theme archive but failed to create a valid database dump. The root cause was that the WordPress application container did not include `mysqldump`, so a `wp db export` command could not complete. The reliable fix was to obtain database connection details from WordPress configuration and run the dump from the dedicated database container using `mariadb-dump` or `mysqldump`.

## Environment

Generic environment:

- WordPress running in Docker.
- MariaDB/MySQL running in a separate database container.
- Backup process creates a Theme/archive file and a SQL dump.
- Backup artifacts are checksum-verified locally and then stored on a separate backup host.
- Remote replication uses restic-compatible repositories.

No real hostnames, domains, IP addresses, credentials or internal paths are included in this public draft.

## Problem

The initial backup attempt looked partially successful because the Theme archive was created, but the SQL export was missing or invalid. In some attempts, the SQL file was absent or 0 bytes. A later explicit check revealed the underlying error:

```text
env: 'mysqldump': No such file or directory
```

This meant the application container could not perform the database export even though WordPress itself was running.

## Current System State / Observed Behavior

Observed behavior:

- Theme archive creation succeeded.
- Database export via `wp db export` failed.
- Missing or 0-byte SQL files were produced during failed attempts.
- Backup could have been falsely considered complete if only the Theme archive was checked.
- Remote replication initially required additional variable-name alignment in the backup script.

## Root Cause

The immediate cause was a missing dump binary (`mysqldump`) in the WordPress application container. In a containerized deployment, the application container is not guaranteed to include database client tools.

A secondary lesson was that reading WordPress configuration by executing or including configuration files may produce unexpected application output in some contexts. For automation, use deterministic configuration access such as WP-CLI config reads or safe text parsing.

## Fix / Resolution

The fix was to run the database export from the database container instead of the WordPress application container.

Generic pattern:

```bash
DB_NAME=$(docker exec wordpress wp config get DB_NAME --allow-root)
DB_USER=$(docker exec wordpress wp config get DB_USER --allow-root)
DB_PASS=$(docker exec wordpress wp config get DB_PASSWORD --allow-root)

docker exec -i \
  -e MYSQL_PWD="$DB_PASS" \
  -e DB_NAME="$DB_NAME" \
  -e DB_USER="$DB_USER" \
  db-container sh -c '
    DUMP_BIN="$(command -v mariadb-dump || command -v mysqldump)"
    "$DUMP_BIN" \
      --single-transaction \
      --quick \
      --routines \
      --triggers \
      --events \
      --default-character-set=utf8mb4 \
      -h 127.0.0.1 \
      -P 3306 \
      -u "$DB_USER" \
      "$DB_NAME"
  ' > wordpress-db-backup.sql
```

Adapt container names, hostnames and credentials handling to your own setup.

## Verification

A backup should not be accepted as valid until all of the following checks pass:

```bash
test -s wordpress-db-backup.sql
head -5 wordpress-db-backup.sql
sha256sum theme-backup.tar.gz wordpress-db-backup.sql > backup.sha256
sha256sum -c backup.sha256
```

Recommended additional checks:

- Verify the backup on the destination system, not only the source.
- Confirm remote restic snapshots exist after replication.
- Reject missing or 0-byte SQL files.
- Add logging and notifications around every step.

## Prevention / Hardening

- Do not rely on the WordPress application container to include database dump tools.
- Prefer dumping from the database container or from a purpose-built backup utility container.
- Use `test -s` to fail on missing or empty SQL files.
- Store Theme/archive, DB dump and checksum together.
- Validate checksums after transfer to the backup host.
- Ensure restic scripts use actual configured environment variable names.
- Add a restore-test process, not just backup creation.

## Lessons Learned

- A WordPress Theme backup is not a complete WordPress restore point.
- Pages, slugs, menus, options and many template assignments are database state.
- Container separation is useful, but backup scripts must respect which container has which tools.
- A backup is not complete until destination validation and remote snapshot verification succeed.

## Disclaimer

This report is a sanitized technical troubleshooting note. Adapt commands and configuration to your own environment. Do not copy firewall, mail, backup, credential, or security commands blindly without understanding their impact.
