---
title: "Traefik served its default cert because the Cloudflare DNS token was valid but not DNS-edit capable"
source_public_report: "incident-reports/infrastructure/2026-05-11_traefik-cloudflare-dns-token-acme.md"
category: "infrastructure"
tags:
  - "traefik"
  - "cloudflare"
  - "letsencrypt"
  - "acme"
  - "dns-01"
  - "selfhosted"
hashtags:
  - "#Traefik"
  - "#Cloudflare"
  - "#LetsEncrypt"
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
    - subreddit: "r/Traefik"
      reason: "Most specific community for Traefik ACME/certificate troubleshooting."
    - subreddit: "r/cloudflare"
      reason: "Relevant because the actual failure is Cloudflare API token scoping/permissions."
    - subreddit: "r/selfhosted"
      reason: "Useful for operators running Dockerized reverse proxies and certificate automation."
    - subreddit: "r/sysadmin"
      reason: "Generalizable lesson about API token scope, monitoring and certificate automation."
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Draft Reddit Post

## Suggested title

Traefik served its default cert because the Cloudflare DNS token was valid but not DNS-edit capable

## Post text

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de

We ran into a Traefik / Let's Encrypt / Cloudflare DNS-01 issue that might be useful for others.

The symptom was simple: the application behind Traefik was reachable, but TLS inspection showed Traefik's fallback certificate instead of a valid Let's Encrypt certificate for the hostname.

Relevant signatures:

```text
subject=CN = TRAEFIK DEFAULT CERT
cloudflare: failed to create TXT record: [status code 403] 10000: Authentication error
```

The misleading part: the Traefik container did have a Cloudflare token variable set. So this was not an obvious "missing env var" case.

The actual issue was token scope/permissions. Several tokens were active, and some could verify or read parts of the account/zone state, but the token available to Traefik could not perform the DNS edit needed for the ACME `_acme-challenge` TXT record.

The fix path:

1. Identify the exact env/secret file Traefik actually loads.
2. Use a dedicated Cloudflare token for Traefik, not a generic automation token.
3. Scope it only to the required zone.
4. Grant the minimal permissions needed for DNS-01: zone read plus DNS edit for that zone.
5. Restart Traefik.
6. Verify that Traefik can issue the certificate and no longer serves the default cert.

The key lesson: a token being present and even "active" does not prove it has the permissions Traefik needs. For DNS-01, the only reliable test is functional: can the token create and delete the required TXT record in the target zone?

We are adding this as a routine check now: verify token, read zone, read DNS records, create/delete a harmless TXT test record, and alert if Traefik ever serves its default cert.

No secrets, hostnames, internal paths or IPs are included here.

## Notes before publication

- Keep `approved: false` until manual review.
- Recommended first subreddit: `r/Traefik`.
- Secondary options: `r/cloudflare`, `r/selfhosted`, `r/sysadmin`.
