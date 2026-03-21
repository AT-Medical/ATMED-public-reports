# ATMED-public-reports

Public reports, published completion summaries, and externally shareable AI-generated documentation for the AT Medical ecosystem.

---

## Repository Structure

```
ATMED-public-reports/
├── reports/
│   ├── issues/          ← Issue completion reports   (ATMED-ISSUE-[AI]-[YYYY]-[XXXX].md)
│   ├── reviews/         ← Review and audit reports   (ATMED-REVIEW-[AI]-[YYYY]-[XXXX].md)
│   └── prs/             ← PR completion reports      (ATMED-PR-[AI]-[YYYY]-[XXXX].md)
└── docs/
    ├── migration/
    │   ├── MIGRATION_GUIDE.md       ← Manual migration instructions
    │   ├── MIGRATION_STATUS.md      ← Current migration status
    │   └── migration_log.json       ← Structured migration log
    └── verification/
        └── report_index_template.json  ← Template for ATMED-private-reports index
```

---

## Naming Convention

All reports in this repository follow the ATMED naming convention:

```
ATMED-[TYPE]-[AI]-[YYYY]-[XXXX].md
```

| Segment  | Description                              | Examples                           |
|----------|------------------------------------------|------------------------------------|
| `TYPE`   | Report type                              | `ISSUE`, `REVIEW`, `PR`            |
| `AI`     | Generating AI system                     | `COPILOT`, `CHATGPT`, `GEMINI`     |
| `YYYY`   | Four-digit year                          | `2026`                             |
| `XXXX`   | Zero-padded sequential ID within year    | `0001`, `0002`, …                  |

---

## Migration Status

This repository is the target for reports migrated from `ATMED-infrastructure`.
See [`docs/migration/MIGRATION_STATUS.md`](docs/migration/MIGRATION_STATUS.md) for the current status.

---

## Verification

Each report published here has a corresponding entry in `AT-Medical/ATMED-private-reports/verification/report_index.json`.
A template for that index is maintained at [`docs/verification/report_index_template.json`](docs/verification/report_index_template.json).

---

<sub style="color: grey; padding-left: 1em; border-left: 3px solid #ccc; display: block;">
Version: 1.0 &nbsp;|&nbsp; Date: 2026-03-21 &nbsp;|&nbsp; Status: active &nbsp;|&nbsp; Repository: ATMED-public-reports
</sub>

---

*© AT Medical GmbH. All rights reserved.*
