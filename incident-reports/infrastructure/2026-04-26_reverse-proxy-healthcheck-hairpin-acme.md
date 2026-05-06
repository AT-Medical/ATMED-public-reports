---
title: "Reverse Proxy Healthcheck False Alert Caused by Hairpin/Self-Access Timeout and ACME Port Routing Conflict"
date: "2026-04-26"
category: "traefik"
tags:
  - "traefik"
  - "docker"
  - "monitoring"
  - "acme"
  - "reverse-proxy"
  - "hairpin-nat"
source_internal_report: "ATMED-private-reports/incident-reports/infrastructure/2026-04-26_atmed-monitoring-hairpin-nat-traefik-mailcow-acme.md"
sanitation_status: "draft"
approval_status: "not-approved"
public_release: false
reddit:
  draft_available: true
  approved: false
  suggested_subreddits:
    - "r/Traefik"
    - "r/selfhosted"
    - "r/docker"
    - "r/sysadmin"
    - "r/homelab"
  selected_subreddit: null
  posted_url: null
---

# Reverse Proxy Healthcheck False Alert Caused by Hairpin/Self-Access Timeout and ACME Port Routing Conflict

## Summary

A self-hosted monitoring setup reported a critical HTTPS healthcheck failure. Initial symptoms looked like a TLS or reverse-proxy outage. After testing local and public access paths separately, the root cause turned out to be a self-access/hairpin routing problem: the server could not reliably connect to its own public IP on ports 80/443, while the reverse proxy itself was healthy locally.

A second issue was found at the same time: ACME HTTP-01 certificate validation failed for one subdomain because port 80 was owned by a different web-facing service instead of the reverse proxy.

## Environment

Generic environment:

- Linux server
- Docker-based services
- Traefik reverse proxy
- self-hosted Healthchecks/Mychecks instance
- mail stack with its own web frontend
- Let's Encrypt ACME HTTP-01 validation
- DNS/proxy layer in front of some hostnames

Real hostnames, domains, IP addresses, check IDs and internal paths have been removed from this public draft.

## Problem

Monitoring reported that an HTTPS heartbeat had not arrived. A parallel certificate/TLS-related warning made the situation look like a wider reverse proxy or TLS incident.

Testing showed:

- DNS resolution worked.
- Direct connection from the server to its own public IP on 80/443 timed out.
- Local access to the reverse proxy via loopback worked.
- Resolving the monitored hostname to `127.0.0.1` worked and returned the expected application response.

## Current System State / Observed Behavior

The reverse proxy was listening and responding locally. The service behind the monitored hostname was reachable when the hostname was explicitly resolved to localhost.

The public self-access path from the same host to its own public IP timed out. This caused the healthcheck to fail even though the local service path was healthy.

At the same time, ACME HTTP-01 validation failed because port 80 was not owned by the reverse proxy. A different service was bound to port 80, while the reverse proxy handled HTTPS and an alternate HTTP mapping.

## Root Cause

There were two related but distinct causes:

1. **Monitoring design issue:** the local heartbeat depended on a connection path through the server's own public IP. That path was unreliable due to self-access/hairpin routing behavior.
2. **ACME routing issue:** HTTP-01 validation expected the reverse proxy to serve the challenge on port 80, but port 80 was bound by another service.

## Fix / Resolution

Short-term workaround:

- Keep testing the local HTTPS/reverse-proxy path.
- Force the monitored hostname to resolve to `127.0.0.1` for the local heartbeat command.
- Avoid using the server's own public IP as the local self-check path.

Generalized example:

```bash
curl -kfsS --resolve monitored.example.com:443:127.0.0.1 https://monitored.example.com/ping/[REDACTED_CHECK_ID] >/dev/null
```

Long-term architecture fix:

- Make the reverse proxy the central entry point for ports 80 and 443.
- Move the other web frontend behind the reverse proxy or keep it on an internal/admin-only alternate port.
- Ensure ACME HTTP-01 challenges consistently reach the reverse proxy.
- Separate internal self-checks from external public monitoring checks.

## Verification

The local workaround was verified by running the heartbeat command with explicit local hostname resolution. The command returned the expected success response.

Additional verification steps:

- Test reverse proxy locally via `https://127.0.0.1`.
- Test the application hostname via `curl --resolve hostname:443:127.0.0.1`.
- Test true external access from an external network or monitoring provider separately.
- Test ACME challenge routing after moving port 80 to the reverse proxy.

## Prevention / Hardening

- Do not treat local self-checks and external uptime checks as the same signal.
- Avoid routing local service checks through the server's own public IP unless hairpin behavior is explicitly known to work.
- Use the reverse proxy as the single owner of ports 80 and 443 where ACME HTTP-01 is used.
- Document which service owns which public port.
- Add a troubleshooting runbook for false alerts caused by loopback/public-path differences.

## Lessons Learned

- A timeout to the server's own public IP does not necessarily mean the service is down.
- `curl --resolve` is useful for separating reverse-proxy/application health from public DNS/routing behavior.
- ACME HTTP-01 validation becomes fragile when port 80 is split across multiple web entry points.
- Monitoring should distinguish between local health, reverse-proxy health, public reachability, and certificate issuance state.

## Disclaimer

This report is a sanitized technical troubleshooting note. Adapt commands and configuration to your own environment. Do not copy firewall, mail, or security rules blindly without understanding their impact.
