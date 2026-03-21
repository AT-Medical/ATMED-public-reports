# Migration Guide — ATMED-infrastructure → ATMED-public-reports

**Migration ID:** ATMED-MIG-2026-0001
**Status:** PENDING — awaiting access to ATMED-infrastructure
**Prepared:** 2026-03-21

---

## Overview

This guide documents the manual steps required to complete the cross-repository migration of all normalised completion reports from `ATMED-infrastructure` into `ATMED-public-reports` (public) and `ATMED-private-reports` (verification index).

The directory structure in `ATMED-public-reports` is ready. The migration itself must be performed manually as `ATMED-infrastructure` was not accessible at the time this guide was prepared.

---

## Prerequisites

- Read access to `AT-Medical/ATMED-infrastructure`
- Write access to `AT-Medical/ATMED-public-reports`
- Write access to `AT-Medical/ATMED-private-reports`
- Local clones of all three repositories

---

## Step 1 — Identify Source Reports in ATMED-infrastructure

Run the following in a local clone of `ATMED-infrastructure` to list all candidate reports:

```bash
find . -type f -name "*.md" | grep -iE "(abschluss|report|review|audit|implementation|completion)" | sort
```

For each file found, determine:

| Check | Criteria |
|-------|----------|
| Naming compliant | Does the filename match `ATMED-[TYPE]-[AI]-[YYYY]-[XXXX].md`? |
| Type identified | Is the type `ISSUE`, `REVIEW`, or `PR`? |
| Content complete | Does the file have meaningful content (not a placeholder)? |
| Relevant | Does it conform to ATMED-Systematik or was it classified as relevant in the audit? |

---

## Step 2 — Classify Each Report

Assign each report one of the three types:

| Type     | Target Directory              | Criteria                                               |
|----------|-------------------------------|--------------------------------------------------------|
| `ISSUE`  | `reports/issues/`             | Relates to a GitHub Issue; completion or setup report  |
| `REVIEW` | `reports/reviews/`            | Audit, code review, or assessment report               |
| `PR`     | `reports/prs/`                | Relates to a GitHub Pull Request                       |

---

## Step 3 — Rename Non-Compliant Files

If a file does not conform to the naming convention `ATMED-[TYPE]-[AI]-[YYYY]-[XXXX].md`, rename it:

```
ATMED-[TYPE]-[AI]-[YYYY]-[XXXX].md
```

Where:

| Segment  | Description                              | Examples                  |
|----------|------------------------------------------|---------------------------|
| `TYPE`   | `ISSUE`, `REVIEW`, or `PR`               | `ISSUE`                   |
| `AI`     | Generating AI system                     | `COPILOT`, `CHATGPT`, `GEMINI` |
| `YYYY`   | Four-digit year from report date         | `2026`                    |
| `XXXX`   | Zero-padded sequential ID within the year| `0001`, `0002`, …         |

**Uniqueness:** Before assigning an ID, check for existing files in the target directory:

```bash
ls reports/issues/ | grep "ATMED-ISSUE-COPILOT-2026-" | sort
```

Use the next available four-digit number.

**Non-destructive:** Keep the original filename recorded in `docs/migration/migration_log.json` under `renamed_files`.

---

## Step 4 — Copy Reports to ATMED-public-reports

For each classified and (if necessary) renamed report, copy it to the correct subdirectory:

| Type     | Target                                  |
|----------|-----------------------------------------|
| `ISSUE`  | `reports/issues/ATMED-ISSUE-[AI]-[YYYY]-[XXXX].md` |
| `REVIEW` | `reports/reviews/ATMED-REVIEW-[AI]-[YYYY]-[XXXX].md` |
| `PR`     | `reports/prs/ATMED-PR-[AI]-[YYYY]-[XXXX].md`       |

**Do not overwrite** existing files. If a conflict is detected (file already exists or same ID), record it in `migration_log.json` under `conflicts` and skip.

```bash
# Example copy for an issue report
cp /path/to/ATMED-infrastructure/original-report.md \
   /path/to/ATMED-public-reports/reports/issues/ATMED-ISSUE-COPILOT-2026-0001.md
```

---

## Step 5 — Update the Migration Log

After copying each file, update `docs/migration/migration_log.json`:

```json
{
  "migrated_files": [
    {
      "id": "ATMED-ISSUE-COPILOT-2026-0001",
      "source_path": "original/path/in/ATMED-infrastructure.md",
      "target_path": "reports/issues/ATMED-ISSUE-COPILOT-2026-0001.md",
      "type": "ISSUE",
      "ai": "COPILOT",
      "date": "2026-01-15",
      "renamed_from": "Abschlussbericht_ATMED-example.md",
      "migrated_at": "2026-03-21T00:00:00Z"
    }
  ]
}
```

---

## Step 6 — Create Entries in ATMED-private-reports/verification/report_index.json

For each successfully migrated report, add an entry to `ATMED-private-reports/verification/report_index.json`.

Use the template at `docs/verification/report_index_template.json` as a reference.

Each entry must contain:

```json
{
  "id": "ATMED-ISSUE-COPILOT-2026-0001",
  "type": "ISSUE",
  "ai": "COPILOT",
  "date": "2026-01-15",
  "verification_code": "ATMED-VRF-XXXXXXXX",
  "hash": "sha256:optional_hash",
  "reference_path": "reports/issues/ATMED-ISSUE-COPILOT-2026-0001.md",
  "source_repository": "ATMED-infrastructure",
  "source_path": "original/path/in/source/repo.md",
  "migrated_at": "2026-03-21T00:00:00Z",
  "status": "migrated"
}
```

**Generate the SHA-256 hash** (optional but recommended):

```bash
sha256sum reports/issues/ATMED-ISSUE-COPILOT-2026-0001.md
```

**Generate a verification code** using the ATMED-VRF format: `ATMED-VRF-` followed by 8 alphanumeric characters unique to this entry.

---

## Step 7 — Duplicate Check

Before finalising, verify no duplicates exist:

```bash
# Check for duplicate IDs in report_index.json (requires jq)
jq '[.reports[].id] | group_by(.) | map(select(length > 1))' \
   ATMED-private-reports/verification/report_index.json

# Check for duplicate filenames in each directory
for dir in reports/issues reports/reviews reports/prs; do
  echo "--- $dir ---"
  ls "$dir" | sort | uniq -d
done
```

---

## Step 8 — Commit and Finalise

1. **ATMED-public-reports:** Commit all new report files and the updated `migration_log.json`
2. **ATMED-private-reports:** Commit the updated `report_index.json`
3. Update `migration_log.json` with final summary counts and set `"status": "COMPLETED"`

---

## Step 9 — Completion Checklist

- [ ] All reports from ATMED-infrastructure identified
- [ ] All reports classified as ISSUE / REVIEW / PR
- [ ] All non-compliant filenames renamed to ATMED convention
- [ ] All reports copied to correct subdirectory in ATMED-public-reports
- [ ] Migration log updated with every migrated, renamed, skipped, and conflicted file
- [ ] All entries added to ATMED-private-reports/verification/report_index.json
- [ ] No duplicate IDs or files
- [ ] migration_log.json status set to `"COMPLETED"`
- [ ] Both repositories committed and pushed

---

## Conflict Resolution

If a file with the same ID or content already exists in the target:

1. Record the conflict in `migration_log.json` under `conflicts`
2. Do **not** overwrite the existing file
3. Notify the maintainer for manual resolution

---

## Naming Cheat-Sheet

| Non-compliant name example            | Compliant target name                      | Type   |
|---------------------------------------|--------------------------------------------|--------|
| `Abschlussbericht_ATMED-assets.md`    | `ATMED-ISSUE-COPILOT-2026-0001.md`         | ISSUE  |
| `review-report-2026.md`              | `ATMED-REVIEW-COPILOT-2026-0001.md`        | REVIEW |
| `PR_completion_report.md`             | `ATMED-PR-COPILOT-2026-0001.md`            | PR     |

---

<sub style="color: grey; padding-left: 1em; border-left: 3px solid #ccc; display: block;">
Version: 1.0 &nbsp;|&nbsp; Date: 2026-03-21 &nbsp;|&nbsp; Status: pending &nbsp;|&nbsp; Repository: ATMED-public-reports
</sub>

---

*© AT Medical GmbH. All rights reserved.*
