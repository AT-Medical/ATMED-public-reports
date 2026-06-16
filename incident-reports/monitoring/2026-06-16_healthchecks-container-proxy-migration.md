---
title: "Migrating a Self-Hosted Healthchecks Container Behind an Existing Nginx Proxy"
date: "2026-06-16"
category: "monitoring"
tags:
  - "healthchecks"
  - "nginx"
  - "docker"
  - "dns"
  - "tls"
source_internal_report: "private internal reference only"
sanitation_status: "draft"
approval_status: "not-approved"
public_release: false
reddit:
  draft_available: true
  approved: false
  suggested_subreddits:
    - "r/selfhosted"
    - "r/sysadmin"
    - "r/devops"
    - "r/docker"
  selected_subreddit: null
  posted_url: null
---

# Migrating a Self-Hosted Healthchecks Container Behind an Existing Nginx Proxy

## Summary

A self-hosted Healthchecks.io instance was migrated from one Linux server to a dedicated monitoring server. The application and database containers were moved successfully, but several misleading errors appeared during routing and verification. The final fix was to route the service through the already existing containerized Nginx proxy rather than attempting to start a separate host-level Nginx service.

## Environment

Generic environment:

- Linux servers
- Docker Compose
- Healthchecks.io container
- PostgreSQL database container
- Existing Nginx reverse proxy container using host networking
- DNS managed by a cloud DNS provider
- TLS certificates issued through Certbot / ACME HTTP-01

No real production hostnames, IP addresses, secrets, or private paths are included in this public report.

## Problem

The service was still running on an older server even though the monitoring role had moved to a dedicated monitoring server. During migration, the team saw:

```text
gzip: stdin: not in gzip format
curl: (56) Recv failure: Connection reset by peer
HTTP/1.1 400 Bad Request
nginx.service is not active, cannot reload
bind() to 0.0.0.0:80 failed (98: Address already in use)
invalid PID number "" in "/run/nginx.pid"
HTTP/2 404
HTTP/2 502
```

## Current System State / Observed Behavior

The application container and database container started on the new server. A local test to `127.0.0.1:<port>` returned `400 Bad Request`. A host-level Nginx start then failed because ports 80 and 443 were already occupied. Later checks showed that the active web routing was actually handled by an existing Nginx proxy container.

DNS had also recently been changed, so some tests temporarily hit stale or cached routing paths.

## Root Cause

There was no single application failure. The confusing symptoms came from several overlapping causes:

1. A fragile transfer attempt produced an invalid compressed stream.
2. The Healthchecks application enforced host validation, so local tests without the correct `Host` header returned `400 Bad Request`.
3. The active proxy was containerized Nginx, not the host `nginx.service`.
4. Port 80/443 were already occupied by the existing proxy process.
5. Resolver cache briefly pointed some tests to the old service location.

## Fix / Resolution

The resolution was:

1. Export the database and application volume/config using file-based artifacts rather than streaming mixed command output through SSH.
2. Import the database and volume/config on the new monitoring server.
3. Bind the Healthchecks app only to localhost on an internal port.
4. Test locally using the expected Host header.
5. Update DNS to point the public name to the new server.
6. Add a new virtual host to the existing Nginx proxy container.
7. Use a webroot-based ACME challenge path for certificate issuance.
8. Enable HTTPS reverse proxying to the local Healthchecks port.
9. Verify public access.
10. Stop and disable the old deployment only after public verification succeeded.

## Verification

Useful verification steps:

```bash
# Check the app with the correct Host header
curl -I -H "Host: health.example.com" http://127.0.0.1:<local-port>/

# Verify public DNS
dig +short A health.example.com
dig +short AAAA health.example.com

# Bypass DNS cache and test the new target directly
curl -I --resolve health.example.com:443:<new-server-ip> https://health.example.com/

# Test public route
curl -I https://health.example.com/
```

A successful Healthchecks login-gated response may be:

```text
HTTP/2 302
location: /accounts/login/
```

## Prevention / Hardening

- Identify the active reverse proxy layer before editing host-level web server services.
- Document whether Nginx runs as a host service, container, or both.
- Use file-based migration artifacts for database and volume moves.
- For Django-based apps, test with the configured Host header.
- After DNS changes, test via public resolvers, direct `--resolve`, IPv4/IPv6 separated curl, and local resolver cache.
- Add renewal hooks that reload the correct proxy layer, especially when Certbot runs on the host but Nginx runs in a container.

## Lessons Learned

- `nginx.service` can be inactive even while Nginx is actively serving traffic from a container.
- `400 Bad Request` can mean host validation, not app failure.
- `Address already in use` should prompt a port-owner audit before restarting services.
- Do not disable the old service until the new public route is verified from multiple paths.

## Disclaimer

This report is a sanitized technical troubleshooting note. Adapt commands and configuration to your own environment. Do not copy firewall, proxy, certificate, or monitoring configuration blindly without understanding its impact.
