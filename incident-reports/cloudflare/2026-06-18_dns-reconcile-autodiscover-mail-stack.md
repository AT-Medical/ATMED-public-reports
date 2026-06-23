---
title: "Making DNS Reconcile Idempotent for Mail Autodiscover Records"
date: "2026-06-18"
category: "cloudflare"
tags:
  - "cloudflare"
  - "dns"
  - "mail"
  - "autodiscover"
  - "mailcow"
  - "automation"
source_internal_report: "private/internal reference only"
sanitation_status: "draft"
approval_status: "not-approved"
public_release: false
reddit:
  draft_available: true
  approved: false
  suggested_subreddits:
    - "r/cloudflare"
    - "r/selfhosted"
    - "r/sysadmin"
  selected_subreddit: null
  posted_url: null
---

# Making DNS Reconcile Idempotent for Mail Autodiscover Records

## Summary

A mail infrastructure maintenance workflow was built to standardize DNS records for multiple mail domains. The goal was to improve mail client auto-configuration by ensuring consistent Autodiscover, Autoconfig, IMAP, SMTP Submission, SPF, DKIM, DMARC, MTA-STS, and TLS-RPT records across several zones.

During implementation, several common automation pitfalls appeared: brittle `jq` filters, insufficient Cloudflare API permissions, duplicate DNS record creation attempts, and confusion between Cloudflare-internal DNS state and public DNS delegation state. The workflow was corrected by making zone matching more robust, handling duplicate-record errors idempotently, and separating API verification from public DNS verification.

## Environment

Generic environment:

- Linux administration host.
- Self-hosted mail platform exposing an API for active mail domains and DKIM data.
- Cloudflare DNS API.
- Shell tooling: Bash, Python, curl, jq, dig.
- Mail client goal: easier IMAP/SMTP setup through Autodiscover and Autoconfig.

No real infrastructure hostnames, domains, IP addresses, credentials, or private paths are included in this report.

## Problem

The original operational goal was to reduce manual account setup work for desktop mail clients. Because the target mail client did not provide a reliable local mass-provisioning interface for arbitrary IMAP/SMTP accounts, the better approach was to improve DNS-based discovery.

The implementation needed to:

- read active mail domains from the mail platform,
- find the matching Cloudflare zones,
- create or update standard mail DNS records,
- clean up conflicting legacy records,
- verify the resulting state.

Several problems occurred:

- A `jq` expression attempted to access `.name` after the pipeline had already produced a string.
- A Cloudflare token had DNS-edit rights but not zone-create rights.
- Re-running the DNS reconcile attempted to create identical SRV records and hit Cloudflare duplicate-record errors.
- A verification filter tried to apply `test()` to a boolean value instead of a string.
- Public DNS checks still showed old values even after Cloudflare API records were correct, because public nameserver delegation was still pointing elsewhere.

## Current System State / Observed Behavior

The automation successfully reached the desired Cloudflare-internal DNS state for the target zones after corrections. The desired mail DNS baseline included:

- MX records pointing to the mail host.
- SPF.
- DKIM.
- DMARC.
- MTA-STS.
- TLS-RPT.
- Autoconfig CNAME.
- Autodiscover CNAME.
- SRV records for Autodiscover, IMAPS, and Submission.

However, public DNS verification must be interpreted separately. Cloudflare API state can be correct while global DNS still returns old records if the domain is not delegated to Cloudflare or propagation is not complete.

## Root Cause

There was no single service outage root cause. The incident was an automation and configuration hardening case with several contributing causes:

1. **Client limitation:** The desired desktop client could not be reliably mass-provisioned locally for all arbitrary IMAP/SMTP mailboxes.
2. **DNS standardization gap:** Some domains did not yet share a consistent mail DNS baseline.
3. **Script robustness gaps:** Early Bash/jq/Python logic did not fully handle subdomain-zone matching and idempotent DNS updates.
4. **Permission mismatch:** The Cloudflare token could edit records but could not create new zones.
5. **Verification ambiguity:** API verification and public DNS verification were initially mixed together.

## Fix / Resolution

The workflow was corrected by:

- using DNS-based client discovery as the primary strategy,
- reading active domains from the mail platform API,
- matching subdomains to the correct parent Cloudflare zone,
- creating standard CNAME/SRV/mail-security records,
- manually creating missing Cloudflare zones where the token lacked zone-create permissions,
- deriving a standard DNS baseline from known-good reference zones,
- treating Cloudflare duplicate-record responses as non-fatal in an idempotent reconcile run,
- separating Cloudflare API verification from public DNS verification.

## Verification

Two verification layers were used:

1. **Cloudflare API verification:** confirms what records exist in Cloudflare.
2. **Public DNS verification:** confirms what external resolvers currently see.

This distinction is important. API verification can pass while public DNS still reflects a previous DNS provider until nameserver delegation is updated and propagated.

## Prevention / Hardening

Recommended hardening measures:

- Make all DNS reconcile scripts idempotent.
- Treat duplicate-record responses as an expected state where appropriate.
- Run dry-run mode before destructive changes.
- Combine write operations with automatic verification in the same workflow.
- Keep DNS baseline definitions in version control.
- Separate token scopes for DNS editing and zone creation deliberately.
- Never store API tokens or credentials in scripts or reports.
- Keep Cloudflare API checks and public DNS checks as separate validation steps.

## Lessons Learned

- For mail client onboarding, good DNS-based discovery can be more practical than trying to automate the client UI.
- DNS reconcile tools must assume that some records already exist.
- API state and public DNS state are different layers and must not be conflated.
- Small jq grouping errors can break operational automation in surprising ways.
- Destructive DNS cleanup should always be preceded by dry-run output and followed by verification.

## Disclaimer

This report is a sanitized technical troubleshooting note. Adapt commands and configuration to your own environment. Do not copy DNS, mail, or security rules blindly without understanding their impact.
