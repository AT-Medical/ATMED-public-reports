---
title: "WordPress canonical host change causing Cloudflare challenge / redirect-loop symptoms"
date: 2026-06-19
category: "cloudflare-wordpress"
classification: "public-redacted"
source: "Redacted internal incident report"
public_safety_review: "required before publication"
contains_real_domains: false
contains_real_ips: false
contains_secrets: false
status: "draft"
tags:
  - "wordpress"
  - "cloudflare"
  - "redirect-loop"
  - "canonical-host"
  - "incident-report"
---

# WordPress canonical host change causing Cloudflare challenge / redirect-loop symptoms

## Summary

A production WordPress site behind Cloudflare became unreachable for end users after a live canonical-host change from the apex domain to the `www` host. Local origin and reverse-proxy tests appeared healthy, but real browsers encountered HTTPS warnings, Cloudflare challenge artifacts, and redirect-loop errors. The site recovered after rolling WordPress back to the previous canonical host and removing hardcoded `WP_HOME` / `WP_SITEURL` constants.

## Environment

- WordPress running in a containerized Apache/PHP environment.
- Cloudflare in front of the site for DNS, HTTPS, WAF/challenge handling, and proxying.
- Reverse proxy between Cloudflare and WordPress origin.
- Wordfence installed in WordPress.

All organization-specific hostnames, IP addresses, paths, tokens, and internal infrastructure details have been removed or generalized.

## Problem

The operational work started with a valid Wordfence finding: a sensitive configuration file was publicly reachable. This was fixed by blocking access to sensitive files at the webserver layer.

A follow-up maintenance step then changed WordPress from the existing apex canonical host to `www` by updating:

```text
siteurl
home
WP_HOME
WP_SITEURL
```

After this change, local tests suggested the site was healthy:

```text
apex -> 301 -> www
www  -> 200
sensitive files -> 403
```

However, real users saw browser-level failures:

```text
connection not secure
too many redirects
Cloudflare challenge token in the URL
```

## Current System State / Observed Behavior

The origin responded correctly during some tests, but browsers still failed. This mismatch happened because local checks did not fully represent the end-user Cloudflare challenge and redirect state.

Relevant indicators included:

```text
cf-mitigated: challenge
__cf_chl_tk
Too many redirects
Connection is not secure
```

## Root Cause

The primary cause was the live canonical-host switch to `www`, especially the addition of hardcoded `WP_HOME` and `WP_SITEURL` constants, without first auditing and aligning Cloudflare challenge and redirect behavior.

The Cloudflare/browser layer was still effectively operating with assumptions from the previous apex-domain setup. The result was a broken interaction among:

1. WordPress canonical redirects,
2. Cloudflare challenge/HTTPS handling,
3. browser-held challenge state,
4. existing redirect behavior.

The initial Wordfence sensitive-file fix was not the cause of the availability issue.

## Fix / Resolution

The recovery steps were:

1. Keep the webserver-sensitive-file protection in place.
2. Roll Cloudflare settings back to the previous known-good public state.
3. Remove temporary bypass or emergency rules created during troubleshooting.
4. Restore the previous WordPress canonical host.
5. Remove active hardcoded `WP_HOME` and `WP_SITEURL` constants.
6. Re-test in real browsers, not only with terminal HTTP clients.

After the WordPress canonical rollback, end-user access recovered.

## Verification

Verification included:

- Sensitive files returned `403 Forbidden`.
- The previous canonical host loaded again in real browsers.
- The user confirmed that the site was reachable after the rollback.

## Prevention / Hardening

Recommended safeguards:

- Do not combine security hardening and canonical-host migration in the same live change window.
- Treat canonical host changes as production changes requiring preflight and rollback.
- Audit Cloudflare redirect, WAF, challenge, and HTTPS settings before changing WordPress canonical host values.
- Avoid hardcoding `WP_HOME` and `WP_SITEURL` unless necessary.
- Test from real browsers, especially Safari/iOS, not only from `curl` or origin-local tests.
- Ensure Cloudflare API tokens have sufficient read-only audit and cache-purge permissions before using automated troubleshooting scripts.
- Avoid DNS-only as a first recovery step for proxied production records unless the rollback path is explicit.

## Lessons Learned

A clean `200 OK` from the origin or local reverse proxy does not prove that a site behind Cloudflare is working for end users. For Cloudflare-backed WordPress sites, the real production path includes browser state, challenge state, and edge rules. Canonical-host changes should therefore be tested as full edge-to-browser flows.

## Disclaimer

This report is a generalized, redacted incident write-up. It omits hostnames, IP addresses, secrets, internal paths, and organization-specific security details. It is provided for operational learning and should not be treated as a universal runbook without adapting it to the local infrastructure.
