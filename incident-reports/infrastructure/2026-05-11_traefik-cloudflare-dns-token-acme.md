# Public Incident Report: Traefik ACME DNS-01 Fails with Cloudflare Token Permissions

## Summary

A self-hosted Traefik reverse proxy delivered its default certificate for one HTTPS route because Let's Encrypt certificate issuance via Cloudflare DNS-01 challenge failed. The root cause was a mismatch between the token Traefik was loading and the DNS permissions required to create `_acme-challenge` TXT records.

## Environment

- Reverse proxy: Traefik
- Certificate authority: Let's Encrypt
- DNS provider: Cloudflare
- ACME method: DNS-01 challenge
- Deployment type: Docker-based self-hosted infrastructure

Organization-specific hostnames, paths, IP addresses and token names have been removed.

## Problem

The affected route was reachable at the application layer, but TLS inspection showed Traefik's fallback/default certificate rather than a valid certificate for the hostname.

Observed error signatures included:

```text
subject=CN = TRAEFIK DEFAULT CERT
cloudflare: failed to create TXT record: [status code 403] 10000: Authentication error
```

## Current System State / Observed Behavior

- Traefik had a Cloudflare token variable set inside the container.
- Multiple candidate token files existed in the environment.
- Some tokens were valid and active, and some could read the zone.
- The token actually available to Traefik did not successfully pass a DNS read/edit test for the required zone.

## Root Cause

The failure was not simply that a token was missing. A token existed, but it did not provide the effective Cloudflare permissions required by Traefik's DNS-01 flow.

For Cloudflare DNS-01 issuance, Traefik needs a token that can at least identify/read the relevant zone and create/update/delete DNS TXT records for that zone. If the wrong token is mounted, if the token is scoped to the wrong zone, or if it only has partial read permissions, ACME issuance fails and Traefik may fall back to its default certificate.

## Fix / Resolution

Recommended remediation:

1. Create or identify a dedicated Cloudflare token for Traefik.
2. Scope it to the single DNS zone needed by Traefik.
3. Grant minimal required permissions: zone read plus DNS edit for that zone.
4. Store the token in the environment file or secret store actually loaded by the Traefik container.
5. Restart Traefik.
6. Watch Traefik logs for ACME DNS-01 success.
7. Verify the served certificate externally.

## Verification

Useful verification checks:

```bash
# Confirm Traefik has a token variable set without printing the token value
docker inspect traefik --format '{{range .Config.Env}}{{println .}}{{end}}' \
  | grep -Ei 'CLOUDFLARE|CF_|ACME|DNS|TOKEN' \
  | sed -E 's/=.*/=SET_OR_MASKED/'

# Check the served certificate
openssl s_client -connect example.example.com:443 -servername example.example.com </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates
```

The key test is functional, not cosmetic: the token must be able to create and remove the required `_acme-challenge` TXT record.

## Prevention / Hardening

- Keep separate tokens for separate purposes: Traefik DNS, WAF automation, reporting, emergency/panic actions.
- Avoid using broad global tokens for routine certificate automation.
- Add a scheduled smoke test that checks whether the Traefik DNS token can read the zone and create/delete a harmless TXT test record.
- Monitor for fallback certificates and ACME errors.
- Document which secret file is actually mounted into the Traefik container.

## Lessons Learned

Seeing a token variable inside a container does not prove that the token is correct. For ACME DNS-01, the only reliable validation is a functional DNS permission test against the target zone.

## Disclaimer

This report is a sanitized technical summary. Hostnames, internal paths, IP addresses, token names and organization-specific details were generalized or removed for security reasons.