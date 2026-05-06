---
title: "Healthcheck false alert: reverse proxy was fine locally, but self-access to public IP timed out"
source_public_report: "incident-reports/infrastructure/2026-04-26_reverse-proxy-healthcheck-hairpin-acme.md"
category: "traefik"
tags:
  - "traefik"
  - "docker"
  - "monitoring"
  - "acme"
  - "reverse-proxy"
  - "hairpin-nat"
hashtags:
  - "#Traefik"
  - "#Docker"
  - "#Monitoring"
  - "#ACME"
  - "#ReverseProxy"
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
    - "r/Traefik"
    - "r/selfhosted"
    - "r/docker"
    - "r/sysadmin"
    - "r/homelab"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

Healthcheck false alert: reverse proxy was fine locally, but self-access to public IP timed out

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/Traefik | Reverse proxy / ACME HTTP-01 routing lesson |
| 2 | r/selfhosted | Common self-hosted monitoring and reverse proxy pitfall |
| 3 | r/docker | Docker port ownership and proxy routing relevance |
| 4 | r/sysadmin | Monitoring signal quality and incident analysis |
| 5 | r/homelab | Practical troubleshooting pattern for home/server labs |

## Mandatory Transparency Disclaimer

```text
Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
```

## Draft Post

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and may be published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

I ran into a technical issue that may be useful for others troubleshooting a similar self-hosted setup.

**Environment**

- Linux server
- Dockerized services
- Traefik as reverse proxy
- self-hosted Healthchecks/Mychecks style monitoring
- another web-facing service also binding a public HTTP port
- Let's Encrypt ACME HTTP-01 validation

No real hostnames, IPs, check IDs, private paths, or customer data are included here.

**Problem**

Monitoring reported that an HTTPS heartbeat had failed. At first glance it looked like a TLS or reverse proxy issue.

But the interesting part was this:

- DNS resolved correctly.
- Accessing the monitored hostname from the same server through the server's public IP timed out.
- Accessing the reverse proxy locally worked.
- Resolving the monitored hostname explicitly to `127.0.0.1` also worked.

So the service and reverse proxy were not actually down. The local self-check was using a problematic public self-access path.

**Root cause**

Two issues overlapped:

1. The local heartbeat depended on the server reaching itself through its own public IP. That failed due to self-access / hairpin routing behavior.
2. ACME HTTP-01 validation was also fragile because port 80 was owned by another service instead of the reverse proxy.

**Fix**

Short-term workaround: force the local healthcheck to resolve the monitored hostname to localhost:

```bash
curl -kfsS --resolve monitored.example.com:443:127.0.0.1 https://monitored.example.com/ping/[REDACTED_CHECK_ID] >/dev/null
```

Long-term fix:

- make Traefik the single public entry point for 80/443;
- move the other web frontend behind Traefik or keep it internal/admin-only;
- ensure ACME HTTP-01 reaches the reverse proxy;
- separate internal self-checks from true external monitoring checks.

**Verification**

The local workaround returned the expected success response. Local reverse proxy checks were healthy, while the public self-access path was the failing component.

**Prevention**

The main lesson: do not treat local self-checks through the server's own public IP as equivalent to external monitoring. Test these as separate signals:

- application health;
- reverse proxy health;
- local self-check path;
- external public reachability;
- ACME/certificate issuance path.

Full write-up: [link to public report]

## Footer / Contact

```text
Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
```

## Review Checklist

- [x] Mandatory transparency disclaimer included
- [x] Website and contact address included
- [ ] Link to full public GitHub report included
- [x] No real IPs
- [x] No real hostnames/domains unless intentionally public
- [x] No secrets/tokens
- [x] No customer/patient/person data
- [x] No internal paths
- [x] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
