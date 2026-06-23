---
title: "DNS reconcile for mail autodiscover: lessons from making Cloudflare updates idempotent"
source_public_report: "incident-reports/cloudflare/2026-06-18_dns-reconcile-autodiscover-mail-stack.md"
category: "cloudflare"
tags:
  - "cloudflare"
  - "dns"
  - "mail"
  - "autodiscover"
  - "automation"
hashtags:
  - "#Cloudflare"
  - "#DNS"
  - "#Mail"
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
    - "r/cloudflare"
    - "r/selfhosted"
    - "r/sysadmin"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

DNS reconcile for mail autodiscover: lessons from making Cloudflare updates idempotent

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/cloudflare | Primary technical domain: Cloudflare DNS API and zone/record automation. |
| 2 | r/selfhosted | Self-hosted mail platforms and multi-domain DNS baselines are relevant to this audience. |
| 3 | r/sysadmin | Operational lessons: idempotent DNS reconcile, token scopes, verification layers. |

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

I ran into a DNS automation issue that might be useful for anyone maintaining multiple mail domains through Cloudflare.

**Environment**

- Linux admin host
- Self-hosted mail platform with an API for active domains/DKIM data
- Cloudflare DNS API
- Bash/Python/curl/jq/dig
- Goal: make desktop mail client setup easier via Autodiscover/Autoconfig and standard mail DNS records

**Problem**

The mail client was not realistically scriptable for mass local provisioning of arbitrary IMAP/SMTP accounts. So the better path was to standardize DNS discovery records across all mail domains.

During the automation, several issues appeared:

- a jq filter tried to access `.name` after the pipeline had already produced a string;
- the Cloudflare token could edit DNS records but not create new zones;
- repeated reconcile runs tried to create identical SRV records;
- a verification jq filter applied `test()` to a boolean;
- Cloudflare API state and public DNS state were initially treated too similarly.

**Root cause**

Not a single outage, but a combination of client limitations, inconsistent DNS baselines, incomplete token permissions, and non-idempotent DNS automation.

**Fix**

The workflow was adjusted to:

- use DNS-based Autodiscover/Autoconfig as the main setup path;
- read active mail domains from the mail platform API;
- correctly match subdomains to their Cloudflare parent zones;
- build a standard DNS baseline from known-good reference zones;
- handle duplicate Cloudflare records as a non-fatal idempotency case;
- separate Cloudflare API verification from public DNS verification.

**Verification**

Two verification layers were used:

1. Cloudflare API check: confirms intended records exist in Cloudflare.
2. Public DNS check: confirms what external resolvers currently see.

That separation matters because Cloudflare records can be correct while public DNS still shows old records if nameserver delegation has not moved yet.

**Prevention**

- Always dry-run destructive DNS cleanup.
- Treat duplicate-record responses as expected in reconcile workflows.
- Put write and verification steps in the same script/block.
- Keep DNS baselines versioned.
- Keep zone-create permissions separate from record-edit permissions deliberately.
- Do not store API tokens in scripts or reports.

Full write-up: `incident-reports/cloudflare/2026-06-18_dns-reconcile-autodiscover-mail-stack.md`

## Review Checklist

- [x] Mandatory transparency disclaimer included
- [x] Website and contact address included
- [x] Link/reference to full public GitHub report included
- [x] No real IPs
- [x] No real hostnames/domains from the incident body
- [x] No secrets/tokens
- [x] No customer/patient/person data
- [x] No internal paths
- [x] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
