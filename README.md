# ATMED Public Reports

<p align="center">
  <img src="https://img.shields.io/badge/class-reports--public-0969da" alt="Class Reports Public">
  <img src="https://img.shields.io/badge/lifecycle-active-1f883d" alt="Lifecycle Active">
  <img src="https://img.shields.io/badge/service-reporting-6f42c1" alt="Service Reporting">
  <img src="https://img.shields.io/badge/visibility-public-1f883d" alt="Visibility Public">
  <img src="https://img.shields.io/badge/rollout-report--only-f2cc60" alt="Rollout report only">
</p>

## Repository Dashboard

| Field | Value |
|---|---|
| Repository | `ATMED-public-reports` |
| Class | `reports-public` |
| Lifecycle | `active` |
| Service | `reporting` |
| Owner team | `governance-team` |
| Technical owner | `devops-team` |
| Visibility | `public` |
| Data class | `public` |
| Deployable | `false` |
| Governance baseline | `AT-Medical-GmbH/github-governance` |
| Central rollout issue | `AT-Medical-GmbH/github-governance#26` |
| Local rollout issue | `AT-Medical-GmbH/ATMED-public-reports#3` |

## Purpose

`ATMED-public-reports` is the public report repository for externally shareable AT Medical GitHub ecosystem summaries.

Reports published here must be suitable for public reading. Internal-only notes and non-redacted drafts belong outside this repository.

## Repository Contract

### Belongs here

- public completion summaries
- public review summaries
- public PR summaries
- externally shareable case-style summaries
- public migration notes
- public report indexes

### Does not belong here

- internal-only report material
- operational details not intended for publication
- service source code
- raw notes
- non-redacted drafts

## Repository Structure

```text
ATMED-public-reports/
├── .atmed/
│   └── repository.yml
├── reports/
│   ├── issues/
│   ├── reviews/
│   └── prs/
├── docs/
│   ├── migration/
│   └── verification/
├── indexes/
└── README.md
```

## Naming Convention

Reports in this repository should follow the ATMED naming convention:

```text
ATMED-[TYPE]-[AI]-[YYYY]-[XXXX].md
```

| Segment | Description | Examples |
|---|---|---|
| `TYPE` | Report type | `ISSUE`, `REVIEW`, `PR` |
| `AI` | Generating AI system | `COPILOT`, `CHATGPT`, `GEMINI` |
| `YYYY` | Four-digit year | `2026` |
| `XXXX` | Sequential ID within year | `0001`, `0002` |

## Publication Rule

Public reports are shareable outputs. External posts, case reports or incident-style summaries should remain drafts until explicitly approved by a human maintainer.

## Quality Gates

| Gate | Status |
|---|---|
| Repository metadata | `introduced` |
| README dashboard | `aligned` |
| Public report structure | `documented` |
| Publication rule | `documented` |
| Deployment | `not applicable` |
| Reporting rollout | `report-only` |

## Migration Status

This repository is the target for public reports migrated from earlier ATMED repositories.

See [`docs/migration/MIGRATION_STATUS.md`](docs/migration/MIGRATION_STATUS.md) for the current status where available.

## Verification

Each public report should be traceable through the ATMED reporting process. Public-facing summaries should not expose internal-only context.

## Governance Notes

The authoritative governance baseline is maintained in `AT-Medical-GmbH/github-governance`.

This repository participates in the pilot rollout in **report-only mode**. Blocking checks should only be introduced after the pilot repositories are aligned and the central governance issue is updated.

---

*© AT Medical GmbH. All rights reserved.*
