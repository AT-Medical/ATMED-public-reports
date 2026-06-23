---
title: "TIL: a manual nightly catch-up script should never auto-trigger reboot timers"
source_public_report: "incident-reports/infrastructure/2026-06-18_nightly-catchup-reboot-and-backup-recovery.md"
category: "monitoring"
tags:
  - "systemd"
  - "backup"
  - "monitoring"
  - "selfhosted"
  - "devops"
hashtags:
  - "#systemd"
  - "#backup"
  - "#monitoring"
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
    - "r/devops"
    - "r/sysadmin"
    - "r/homelab"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

TIL: a manual nightly catch-up script should never auto-trigger reboot timers

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/selfhosted | Most relevant for homelab/self-hosted maintenance automation and backup routines. |
| 2 | r/devops | Relevant for runbook design, catch-up jobs, monitoring and safe automation patterns. |
| 3 | r/sysadmin | Relevant for operational recovery, systemd timers and maintenance safeguards. |
| 4 | r/homelab | Useful if framed as a learning from a self-managed multi-server environment. |

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

I ran into a self-hosting / ops issue that may be useful for anyone maintaining nightly jobs with systemd timers.

**Environment**

- Linux servers managed from a central control host
- systemd timers/services for nightly maintenance
- Docker-based services
- restic-style backup verification
- monitoring dashboard that aggregates service status

**Problem**

After a period of degraded reachability, I wanted to manually catch up missed nightly routines: host checks, backups, cleanup, updates and reporting.

The initial catch-up logic was too broad. It selected matching timer-derived services and accidentally included the weekly reboot job. That caused pending shutdown notices and interrupted a live maintenance session.

**Root cause**

The catch-up command used discovery-style matching instead of a strict allowlist. Reboot/shutdown jobs were not explicitly excluded.

**Fix**

The reboot was allowed to complete. After reconnecting, only the intended daily maintenance tasks were rerun manually. Backup recovery was verified through successful snapshot creation, repository checks and mini-restore tests.

**Prevention**

- Make manual catch-up scripts allowlist-based.
- Never include reboot/shutdown jobs unless explicitly requested.
- Keep backup verification separate from update/reporting diagnostics.
- Make dashboard aggregate failures granular enough to show what is actually still broken.

Full write-up: incident-reports/infrastructure/2026-06-18_nightly-catchup-reboot-and-backup-recovery.md

## Review Checklist

- [x] Mandatory transparency disclaimer included
- [x] Website and contact address included
- [x] No real IPs
- [x] No real internal hostnames/domains
- [x] No secrets/tokens
- [x] No customer/patient/person data
- [x] No internal paths
- [x] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
