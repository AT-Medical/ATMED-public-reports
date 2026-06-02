---
title: "Mailcow Rspamd Hyperscan cache errors after restart"
date: "2026-06-02"
category: "mail"
tags:
  - "mailcow"
  - "rspamd"
  - "hyperscan"
  - "selfhosted"
  - "docker"
classification: "public-sanitized"
source_private_report: "ATMED-private-reports/incident-reports/mail/2026-06-02_atmed-mailcow-rspamd-hyperscan-freeze.md"
reddit_draft: "reddit-drafts/mail/2026-06-02_mailcow-rspamd-hyperscan-cache-errors.md"
status: "draft"
approved_for_publication: false
---

# Mailcow Rspamd Hyperscan cache errors after restart

## Summary

A Mailcow installation running Rspamd inside Docker repeatedly logged Hyperscan cache errors after restarts. The recurring log signature was:

```text
rspamd_re_cache_load_hyperscan: bad hs database ... error code: -6
```

Removing generated `.hs` and `.hsmp` files did not provide a durable fix because Rspamd recreated cache files on startup. The stable mitigation was to disable Hyperscan in Rspamd and verify the actual mailflow through queue, container, log and local delivery checks.

## Environment

- Mail stack: Mailcow
- Spam filtering: Rspamd
- Runtime: Docker / Docker Compose
- Affected component: Rspamd Hyperscan cache
- Public report status: sanitized and generalized

## Problem

Rspamd emitted many startup warnings about invalid Hyperscan database files. The files were located in Rspamd's runtime data directory and used `.hs` / `.hsmp` extensions.

A simple cache purge removed the files temporarily, but new files were generated again after Rspamd started. Depending on the host/container combination, the new databases could again be reported as invalid.

## Current System State / Observed Behavior

Observed behavior:

- Rspamd started but logged Hyperscan database load errors.
- Cache files were recreated after deletion.
- Rspamd configuration syntax remained valid.
- Mail queue and delivery were not necessarily affected.
- The key operational question was whether the errors continued in steady state or affected message processing.

## Root Cause

The most likely cause was a host/container/cache compatibility issue affecting Rspamd's generated Hyperscan database files. Cache deletion alone did not address the underlying compatibility issue because the cache was regenerated.

## Fix / Resolution

The stable mitigation was to disable Hyperscan in Rspamd:

```text
disable_hyperscan = true;
```

This setting should be placed in the appropriate Rspamd local configuration file for the Mailcow installation and then verified from inside the container.

Recommended verification:

```bash
rspamadm configtest
rspamadm configdump options | grep -i hyperscan
```

Expected effective configuration:

```text
disable_hyperscan = true;
```

## Verification

A successful recovery should not be judged only by startup logs. The following checks are more meaningful:

1. Rspamd configuration validates successfully.
2. The effective Rspamd config contains `disable_hyperscan = true`.
3. The mail queue is empty or processing normally.
4. Postfix and Dovecot containers are healthy/running.
5. Rspamd logs show no continuing critical errors in steady state.
6. Watchdog/health checks are clean.
7. A local test email is accepted, scanned and delivered.

## Prevention / Hardening

- Treat cache purge as a cleanup step, not as a guaranteed fix.
- Verify steady state after a waiting period, not just immediately after restart.
- Keep a known-good backup after a green system state.
- Re-evaluate Hyperscan only after image, package or host updates.
- Document the decision if disabling Hyperscan is used as a stable mitigation.

## Lessons Learned

- Hyperscan errors can look alarming, but the real operational question is whether Rspamd and mail delivery remain stable.
- If generated Hyperscan database files repeatedly fail to load, disabling Hyperscan can be a reasonable stability-first mitigation.
- Always validate mailflow with queue and delivery checks before declaring the system recovered.

## Disclaimer

This report is a sanitized technical summary of a real troubleshooting process. It intentionally omits organization-specific hostnames, domains, IP addresses, internal paths, customer data and security-sensitive details. Adapt commands and configuration paths to your own Mailcow installation.
