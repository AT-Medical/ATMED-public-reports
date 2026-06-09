# Public Incident Report — Reverse-proxy route returns 404 after container recreate

## Summary

A self-hosted web application running in Docker Compose behind Traefik briefly returned `404 page not found` after the application container was recreated. The container itself was healthy and still had the expected proxy labels. Restarting the reverse proxy restored the route.

## Environment

Generalized environment:

- Web application in Docker Compose
- Traefik reverse proxy
- Docker provider based routing
- TLS-enabled public route
- Security-relevant application configuration changed through environment settings

## Problem

After applying a configuration change and recreating the application container, the public route returned:

```text
404 page not found
```

At the same time, the application container was healthy and connected to the expected Docker network.

## Current System State / Observed Behavior

- Compose service name and container name differed.
- Initial command used the container name instead of the Compose service name.
- Recreate with the correct Compose service succeeded.
- Reverse-proxy labels were present and correct.
- The reverse proxy still returned a 404 until it was restarted.

## Root Cause

The most likely cause was that the reverse proxy did not apply the updated Docker provider routing state after the container recreate. Since labels and container health were correct, a proxy restart was sufficient.

## Fix / Resolution

- Used the correct Compose service name for container recreation.
- Verified Docker labels on the container.
- Restarted the reverse proxy.
- Verified that the application configuration endpoint returned JSON again.
- Verified that the intended security configuration was active.

## Verification

Successful verification included:

```text
application API reachable
intended security setting active
container healthy
reverse proxy route working
```

## Prevention / Hardening

- Use Compose service names, not container names, for `docker compose` commands.
- After a container recreate, verify both application container health and external reverse-proxy routing.
- Treat route verification as a separate control step after container recreation.

## Lessons Learned

A healthy container does not necessarily mean the external route is healthy. Reverse-proxy state should be verified separately after container recreation.

## Disclaimer

This report is a public, sanitized summary. Hostnames, domains, internal paths, credential material and organization-specific security details have been generalized or removed.
