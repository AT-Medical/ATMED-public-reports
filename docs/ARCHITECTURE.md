# ATMED Public Reports – Architecture

## Purpose

`ATMED-public-reports` is the repository for externally shareable and publication-suitable reports within the AT Medical ecosystem.

## Repository Type

This is a **reports / publication repository**, not a runtime service.

## Platform Role

```text
Internal report generation
  ↓
ATMED-public-reports
  ↓
Curated public-facing reports and completion summaries
```

## Scope

- public reports
- published completion summaries
- externally shareable AI-generated documentation
- migration and verification references

## VPS Role

No runtime stack required. Optional mirror path for archival access:

| Purpose | Path |
|---|---|
| Repository sync target | `/srv/atmed/repos/ATMED-public-reports` |

## Relationship to Other Repositories

- `ATMED-private-reports` = internal/private report layer
- `ATMED-public-reports` = public/shareable report layer

## Target State

Conformant when publication role and non-runtime nature are explicitly documented.
