---
title: "WordPress Docker backup looked successful, but the DB dump was missing because mysqldump was not in the app container"
source_public_report: "incident-reports/backup/2026-06-23_wordpress-database-export-backup-troubleshooting.md"
category: "backup"
tags:
  - "wordpress"
  - "docker"
  - "mariadb"
  - "backup"
  - "restic"
hashtags:
  - "#WordPress"
  - "#Docker"
  - "#Backup"
  - "#MariaDB"
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
    - "r/docker"
    - "r/sysadmin"
    - "r/homelab"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

WordPress Docker backup looked successful, but the DB dump was missing because mysqldump was not in the app container

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/selfhosted | Most relevant for self-hosted WordPress, Docker and backup workflows. |
| 2 | r/docker | Relevant because the root cause is container responsibility/tooling separation. |
| 3 | r/sysadmin | Useful operational backup lesson: validate artifacts and destination backups. |
| 4 | r/homelab | Relevant for Dockerized backup and restore workflows. |

## Mandatory Transparency Disclaimer

```text
Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
```

## Draft Post

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de

I ran into a WordPress/Docker backup issue that may be useful for others.

**Environment**

- WordPress in an application container
- MariaDB/MySQL in a separate database container
- Backup flow intended to create both a Theme/archive file and a SQL dump
- Backup artifacts were later checksum-verified and replicated remotely

**Problem**

The backup initially looked half-successful because the Theme archive was created. But the database dump was missing or invalid. The key error was:

```text
env: 'mysqldump': No such file or directory
```

So `wp db export` failed because `mysqldump` was not available inside the WordPress application container.

**Root cause**

The application container was not a database-client container. It could run WordPress and WP-CLI, but it did not include the dump binary needed by `wp db export`.

**Fix**

Instead of dumping from the WordPress app container, read the DB settings through WP-CLI and run the dump from the actual database container using `mariadb-dump` or `mysqldump`.

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
    "$DUMP_BIN" --single-transaction --quick --routines --triggers --events \
      --default-character-set=utf8mb4 \
      -h 127.0.0.1 -P 3306 -u "$DB_USER" "$DB_NAME"
  ' > wordpress-db-backup.sql
```

**Verification**

Do not treat the backup as valid until the SQL file exists, is non-zero, has a plausible dump header, and passes checksum validation both before and after transfer:

```bash
test -s wordpress-db-backup.sql
head -5 wordpress-db-backup.sql
sha256sum theme-backup.tar.gz wordpress-db-backup.sql > backup.sha256
sha256sum -c backup.sha256
```

**Prevention**

- Do not assume the app container has DB client tools.
- Prefer dumping from the DB container or a dedicated backup container.
- Fail closed on missing/0-byte SQL files.
- Verify backups on the destination, not only the source.
- Confirm remote snapshots if using restic or a similar system.

Full write-up: `incident-reports/backup/2026-06-23_wordpress-database-export-backup-troubleshooting.md`

## Footer / Contact

```text
Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
```

## Review Checklist

- [x] Mandatory transparency disclaimer included
- [x] Website and contact address included
- [x] Link/reference to full public GitHub report included
- [x] No real IPs
- [x] No real internal hostnames/domains
- [x] No secrets/tokens
- [x] No customer/patient/person data
- [x] No internal paths
- [x] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
