# Incident Reports

This directory contains sanitized public incident and troubleshooting reports derived from internal ATMED AI Brain incident reports.

## Purpose

These reports are intended to help other users who encounter similar technical problems while ensuring that no AT Medical-specific confidential information is published.

## Mandatory Rule

Only sanitized reports may be stored here. Do not include:

- real internal hostnames
- real internal domains unless explicitly intended for publication
- real public IP addresses from AT Medical infrastructure
- secrets, tokens, credentials, private keys
- customer data
- patient data
- personal data
- internal security architecture details
- internal file paths unless generalized
- raw log excerpts containing identifying information

## Recommended Structure

```text
incident-reports/
├── mail/
├── docker/
├── cloudflare/
├── traefik/
├── linux/
├── monitoring/
└── other/
```

## Report Flow

```text
Internal AI Brain report
  -> sanitized public draft
  -> review
  -> public report in this repository
  -> optional Reddit draft/post
```

## Canonical Template

Use:

```text
templates/PUBLIC_INCIDENT_REPORT_TEMPLATE.md
```
