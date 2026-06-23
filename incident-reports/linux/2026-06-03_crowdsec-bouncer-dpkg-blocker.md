---
title: "CrowdSec firewall bouncer blocks dpkg during Ubuntu package upgrade"
date: "2026-06-03"
category: "linux"
tags:
  - "linux"
  - "ubuntu"
  - "apt"
  - "dpkg"
  - "crowdsec"
  - "systemd"
  - "nftables"
source_internal_report: "internal reference only"
sanitation_status: "draft"
approval_status: "not-approved"
public_release: false
reddit:
  draft_available: true
  approved: false
  suggested_subreddits:
    - "r/linuxadmin"
    - "r/sysadmin"
    - "r/selfhosted"
  selected_subreddit: null
  posted_url: null
---

# CrowdSec firewall bouncer blocks dpkg during Ubuntu package upgrade

## Summary

During a routine Ubuntu server upgrade, `dpkg` became stuck because the `crowdsec-firewall-bouncer-nftables` package failed during its post-installation step. The package attempted to start its systemd service, but the service exited with a runtime error. As a result, `apt` could not complete the upgrade transaction.

The practical recovery path was to treat the firewall bouncer as the failed component, temporarily isolate it from the package transaction, restore a clean `dpkg` state, and then evaluate whether to reinstall or replace the bouncer separately.

## Environment

Generic environment:

- Ubuntu LTS server
- `apt` / `dpkg` package management
- systemd service supervision
- CrowdSec installed
- `crowdsec-firewall-bouncer-nftables` used for firewall enforcement
- Kernel and container/runtime packages included in the broader upgrade

No real hostnames, domains, IP addresses, usernames, or private internal paths are included in this public draft.

## Problem

The upgrade failed while configuring the firewall bouncer package. The visible package-manager symptoms were:

```text
Errors were encountered while processing:
 crowdsec-firewall-bouncer-nftables
needrestart is being skipped since dpkg has failed
Sub-process /usr/bin/dpkg returned an error code (1)
```

The service itself showed a more specific runtime failure:

```text
process terminated with error: bouncer stream halted
```

A subtle but important detail: the systemd `ExecStartPre` configuration test succeeded, but the real `ExecStart` failed. In other words, a syntactically valid configuration did not mean the bouncer could actually run.

## Current System State / Observed Behavior

Observed behavior:

- `apt full-upgrade` started normally.
- Multiple packages upgraded successfully.
- A new kernel was installed.
- `dpkg` failed when configuring the CrowdSec firewall bouncer package.
- The bouncer systemd service repeatedly failed and restarted.
- The package manager stayed in an incomplete state.
- `needrestart` did not continue because `dpkg` had failed.

The service was effectively in a restart loop, while `dpkg` was blocked by the package's post-installation behavior.

## Root Cause

The immediate root cause was not the whole Ubuntu upgrade. It was the firewall bouncer package's post-installation step failing because its systemd service could not start successfully.

The likely runtime issue was that the bouncer could not establish or maintain its expected decision stream / local CrowdSec communication path, causing:

```text
bouncer stream halted
```

The exact environment-specific reason must be diagnosed locally, but the operational lesson is broader: packages that require a service to start during post-install can block the whole package manager if that service is broken.

## Fix / Resolution

A safe, generalized recovery sequence is:

1. Stop and disable the failing service, if it exists.
2. Reset the failed systemd state.
3. If normal package repair still fails, temporarily isolate or remove the broken bouncer package from the active package transaction.
4. Repair package-manager state.
5. Finish upgrades.
6. Verify that `dpkg --audit` is clean.
7. Reboot only after package state is consistent, especially if a kernel was upgraded.
8. Reinstall or replace the bouncer only after the underlying CrowdSec/API/firewall issue has been diagnosed.

Use any package-removal or purge operation only when you understand the security tradeoff: disabling a firewall bouncer may reduce local automated blocking until it is replaced or reinstalled.

## Verification

After recovery, verify package state:

```bash
dpkg --audit
```

Expected result: no output.

Check failed services:

```bash
systemctl --failed --no-pager
```

Check whether a reboot is required:

```bash
test -f /var/run/reboot-required && cat /var/run/reboot-required || echo "No reboot required."
```

If the bouncer will be reinstalled, verify the CrowdSec core service and bouncer registration first:

```bash
systemctl status crowdsec --no-pager -l || true
cscli bouncers list || true
```

## Prevention / Hardening

- Monitor systemd restart loops and alert on unusually high restart counters.
- Include `dpkg --audit` in server update workflows.
- Maintain a documented recovery path for broken security agents/bouncers.
- Reinstall security enforcement only after confirming the underlying local API and firewall runtime behavior.

## Lessons Learned

- A package can block `dpkg` because its post-install step depends on a successful systemd service start.
- `ExecStartPre` success does not guarantee runtime success.
- Repair the package manager first, reboot second, and reinstall the broken enforcement component third.
- Security tooling must be recoverable; otherwise, the security layer itself can become an availability risk.

## Disclaimer

This report is a sanitized technical troubleshooting note. Adapt commands and configuration to your own environment. Do not copy firewall or security-agent changes blindly without understanding their impact.
