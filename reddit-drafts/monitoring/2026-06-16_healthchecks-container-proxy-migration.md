---
title: "Healthchecks.io behind Docker/Nginx migration: 400 Bad Request, inactive nginx.service, and stale DNS"
source_public_report: "incident-reports/monitoring/2026-06-16_healthchecks-container-proxy-migration.md"
category: "monitoring"
tags:
  - "healthchecks"
  - "nginx"
  - "docker"
  - "dns"
  - "tls"
hashtags:
  - "#SelfHosted"
  - "#DevOps"
  - "#Docker"
  - "#Nginx"
  - "#Monitoring"
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
    - "r/sysadmin"
    - "r/devops"
    - "r/docker"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

Healthchecks.io Docker migration: when `400 Bad Request`, inactive `nginx.service`, and DNS cache all point in different directions

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/selfhosted | Healthchecks.io, Docker, Nginx and self-hosted monitoring are a direct fit. |
| 2 | r/sysadmin | The incident includes DNS, TLS, service ownership and operational runbook lessons. |
| 3 | r/devops | Useful for migration, verification and observability workflows. |
| 4 | r/docker | Containerized Nginx and application migration details are relevant. |

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

I ran into a monitoring migration issue that may be useful for others running self-hosted Healthchecks.io behind Docker/Nginx.

**Environment**

- Linux servers
- Docker Compose
- Healthchecks.io + PostgreSQL
- Existing containerized Nginx reverse proxy
- DNS + ACME/TLS

**Problem**

During a move from one server to a dedicated monitoring host, several errors appeared that looked unrelated at first:

```text
gzip: stdin: not in gzip format
curl: (56) Recv failure: Connection reset by peer
HTTP/1.1 400 Bad Request
nginx.service is not active, cannot reload
bind() to 0.0.0.0:80 failed (98: Address already in use)
invalid PID number "" in "/run/nginx.pid"
HTTP/2 404 / 502 during DNS transition
```

**Root cause**

There were multiple overlapping issues:

- A streamed tar transfer was fragile and produced an invalid archive.
- Healthchecks/Django returned `400` because local tests did not use the configured Host header.
- The active proxy was not the host `nginx.service`; it was an Nginx container running with host networking.
- DNS and local resolver cache briefly pointed to different places.

**Fix**

The final resolution was:

1. Export/import DB and volumes through real files, not mixed stdout streams.
2. Start Healthchecks on the new host bound to localhost.
3. Test with the correct `Host` header.
4. Add the vhost to the existing proxy container instead of starting a second host Nginx.
5. Use ACME webroot for the new vhost.
6. Verify with public DNS, direct `--resolve`, IPv4-only curl and local resolver checks.
7. Disable the old deployment only after the new public route worked.

**Verification**

A successful login-gated Healthchecks response looked like:

```text
HTTP/2 302
location: /accounts/login/
```

**Prevention**

The biggest takeaway: before touching `nginx.service`, confirm who actually owns ports 80/443. Containerized proxies can make systemd status misleading.

Full write-up: [link to public report after approval]

## Review Checklist

- [x] Mandatory transparency disclaimer included
- [x] Website and contact address included
- [ ] Link to full public GitHub report inserted after publication approval
- [x] No real IPs
- [x] No real hostnames/domains except organization footer
- [x] No secrets/tokens
- [x] No customer/patient/person data
- [x] No internal paths
- [x] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
