---
title: "Restic check failures and stale locks in a self-hosted backup control plane"
source_public_report: "incident-reports/infrastructure/2026-05-11_restic-check-stale-lock-backup-control-plane.md"
category: "infrastructure"
tags:
  - "backup"
  - "restic"
  - "rclone"
  - "selfhosted"
  - "devops"
hashtags:
  - "#backup"
  - "#restic"
  - "#selfhosted"
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
    - "r/sysadmin"
    - "r/devops"
    - "r/homelab"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Reddit Draft

## Suggested title

Restic check failures and stale locks in a self-hosted backup control plane: lessons learned

## Suggested subreddits and rationale

- `r/selfhosted`: best fit for a self-hosted backup dashboard/control-plane setup.
- `r/sysadmin`: relevant because the issue concerns production-style backup verification and operational recovery.
- `r/devops`: relevant for CI, change management, systemd automation and reliability engineering.
- `r/homelab`: relevant if framed as a lab-to-production learning story, but less enterprise-focused.

## Draft text

I recently worked through an incident in a self-hosted backup control plane built around restic, rclone, systemd and a web dashboard.

The interesting part: backup snapshots were created successfully, but the overall run was still marked as failed because the later repository verification phase returned a non-zero exit code. Instead of being recorded as a structured `check failed` status, shell error handling surfaced it as a generic line-number failure.

A second issue involved a stale restic repository lock after an interrupted or long-running operation. The correct remediation was not to blindly unlock, but first to verify that no backup, restic or rclone process was still active, and only then remove the stale lock.

Main lessons learned:

- Snapshot creation and verification should be tracked separately.
- `restic check` failures should be first-class operational states, not generic script crashes.
- Global shell traps can hide the actual failing command if expected non-zero states are not handled explicitly.
- Dashboard/UI changes should be separated from backup execution logic.
- Direct live patching of backup scripts is risky; version control and reviewable changes matter.
- A backup system is not enterprise-ready until restore has been tested and documented.

The remediation path was to restore the last syntactically valid script state, verify endpoint connectivity, inspect active processes, remove stale locks only after confirmation, and move the project into a proper repository with runbooks and planned CI checks.

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de

## Publication status

Draft only. Not approved for publication.
