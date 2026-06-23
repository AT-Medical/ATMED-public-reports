---
title: "Manual Nightly Catch-up After Outage Accidentally Triggered a Scheduled Reboot"
date: "2026-06-18"
category: "monitoring"
tags:
  - "linux"
  - "systemd"
  - "backup"
  - "monitoring"
  - "devops"
source_internal_report: "Internal AT Medical report, redacted for public release"
sanitation_status: "draft"
approval_status: "not-approved"
public_release: false
reddit:
  draft_available: true
  approved: false
  suggested_subreddits:
    - "r/selfhosted"
    - "r/devops"
    - "r/sysadmin"
    - "r/homelab"
  selected_subreddit: null
  posted_url: null
---

# Manual Nightly Catch-up After Outage Accidentally Triggered a Scheduled Reboot

## Summary

A self-hosted infrastructure environment had missed scheduled nightly maintenance after a period of reduced reachability. A broad manual catch-up command was used to trigger health checks, backups, cleanup jobs, update checks and reporting. Because the catch-up logic selected services too broadly, it also triggered a scheduled weekly reboot job. The reboot was allowed to complete. Afterward, host reachability, backup completion, repository checks and mini-restore tests were verified successfully. Two non-critical routine modules still required follow-up: remote update checks and daily briefing/report delivery.

## Environment

Generic environment:

- Linux servers managed through a central control host.
- `systemd` services and timers for nightly operations.
- Containerized services using Docker / Docker Compose.
- Backup orchestration using a backup control host and restic-compatible remote repositories.
- Monitoring/status dashboard with aggregate health state.

Organization-specific hostnames, domains, IP addresses, internal paths and credentials have been removed.

## Problem

The operator wanted to recover missed nightly routines after several days of degraded reachability. The first catch-up command was too permissive and started all matching timer-derived services, including a weekly reboot/shutdown job. This caused pending shutdown notices and interrupted a live SSH/backup session.

## Current System State / Observed Behavior

Observed behavior included:

```text
System is going down. Unprivileged users are not permitted to log in anymore.
```

After the reboot completed, all hosts were reachable again. Core nightly tasks such as system checks, backups and cleanup completed. Backup logs showed successful snapshots, repository checks and mini-restore tests.

Remaining non-green states were limited to remote update checks and the briefing/reporting job.

## Root Cause

The immediate root cause was overly broad catch-up service selection. Instead of using an explicit allowlist of daily catch-up tasks, the command matched and started a weekly reboot service as well.

Contributing factors:

- reboot/shutdown jobs were not explicitly excluded;
- catch-up logic was based on service discovery rather than a fixed safe task list;
- the monitoring dashboard summarized remaining task failures as an overall failed state;
- update/reporting failures were separate issues and should not be conflated with backup recovery.

## Fix / Resolution

The reboot was allowed to complete. After reconnecting, the operator manually started only the intended daily maintenance components and verified:

- host reachability;
- system checks;
- backup completion;
- repository integrity checks;
- mini-restore tests;
- cleanup completion.

The remaining update/reporting failures were isolated for follow-up instead of re-running broad catch-up logic again.

## Verification

Recovery was verified through:

- host availability checks returning OK;
- successful systemd status for core nightly routines;
- backup logs showing successful snapshots;
- restic repository checks returning OK;
- mini-restore tests returning OK.

## Prevention / Hardening

Recommended prevention measures:

- use explicit allowlists for manual catch-up routines;
- never auto-include reboot/shutdown services in catch-up scripts;
- separate backup recovery verification from update/reporting diagnostics;
- make dashboard aggregate failures more granular;
- document a dedicated runbook for missed nightly routines after outages.

## Lessons Learned

- Manual recovery scripts should be conservative and allowlist-based.
- A successful backup job is not enough; repository checks and restore tests are the stronger verification signal.
- Scheduled reboot jobs should require explicit operator intent in catch-up scenarios.
- A dashboard-level failed state can be operationally acceptable if the remaining failure domain is clearly isolated.

## Disclaimer

This report is a sanitized technical troubleshooting note. Adapt commands and configuration to your own environment. Do not copy maintenance, backup, firewall, mail or security rules blindly without understanding their impact.
