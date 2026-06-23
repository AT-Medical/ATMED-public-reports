---
title: "Ubuntu upgrade stuck because CrowdSec firewall bouncer post-install failed"
source_public_report: "ATMED-public-reports/incident-reports/linux/2026-06-03_crowdsec-bouncer-dpkg-blocker.md"
category: "linux"
tags:
  - "linux"
  - "ubuntu"
  - "apt"
  - "dpkg"
  - "crowdsec"
  - "systemd"
  - "nftables"
hashtags:
  - "#Linux"
  - "#Ubuntu"
  - "#CrowdSec"
  - "#systemd"
  - "#SelfHosted"
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
    - "r/linuxadmin"
    - "r/sysadmin"
    - "r/selfhosted"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

Ubuntu upgrade stuck because CrowdSec firewall bouncer post-install failed

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/linuxadmin | Best technical fit for Ubuntu package-manager and systemd recovery. |
| 2 | r/sysadmin | Useful for admins dealing with failed post-install scripts during maintenance windows. |
| 3 | r/selfhosted | Relevant because CrowdSec and firewall bouncers are common in self-hosted server stacks. |

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

I ran into a server maintenance issue that may be useful for others troubleshooting Ubuntu upgrades with CrowdSec installed.

**Environment**

- Ubuntu LTS server
- `apt` / `dpkg`
- systemd
- CrowdSec with the nftables firewall bouncer
- Broader upgrade included kernel and container/runtime packages

No real hostnames, IPs, domains, usernames, or private paths are included here.

**Problem**

During `apt full-upgrade`, the package manager got stuck while configuring `crowdsec-firewall-bouncer-nftables`.

The high-level symptom was a failed `dpkg` transaction. The more interesting service-level error was:

```text
process terminated with error: bouncer stream halted
```

The systemd `ExecStartPre` test succeeded, but the actual service start failed. So the configuration could be syntactically valid while the real runtime behavior still failed.

**Root cause**

The immediate problem was that the bouncer package's post-install step depended on starting a systemd service that could not actually run. That blocked `dpkg` and prevented the upgrade transaction from completing.

**Fix approach**

The safe recovery pattern was:

1. Stop/disable the failing service where possible.
2. Reset failed systemd state.
3. If normal package repair still fails, isolate the broken bouncer from the package transaction.
4. Repair package-manager state.
5. Finish upgrades.
6. Verify `dpkg --audit` is clean.
7. Reboot only after package state is consistent.
8. Diagnose and reinstall/replace the bouncer separately.

Important caveat: disabling or removing a firewall bouncer can reduce local automated blocking. Treat that as a temporary package-manager recovery step, not as a security posture decision.

**Verification**

The important checks afterward are:

```bash
dpkg --audit
systemctl --failed --no-pager
```

Then check whether a reboot is required, especially if a new kernel was installed.

**Prevention**

- Alert on systemd services with extreme restart loops.
- Include `dpkg --audit` in maintenance workflows.
- Keep a runbook for failed post-install scripts of security agents/bouncers.
- Reinstall enforcement components only after confirming the underlying CrowdSec/API/firewall runtime works.

Full write-up: ATMED-public-reports/incident-reports/linux/2026-06-03_crowdsec-bouncer-dpkg-blocker.md

## Footer / Contact

```text
Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
```

## Review Checklist

- [x] Mandatory transparency disclaimer included
- [x] Website and contact address included
- [ ] Link to full public GitHub report included after publication URL exists
- [x] No real IPs
- [x] No real hostnames/domains except organization footer
- [x] No secrets/tokens
- [x] No customer/patient/person data
- [x] No internal paths
- [x] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
