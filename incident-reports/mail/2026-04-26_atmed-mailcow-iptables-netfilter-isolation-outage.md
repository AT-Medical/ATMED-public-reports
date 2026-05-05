# Public Incident Report – Self-Hosted Mail Inbound Disruption

**Report type:** Public / Sanitized / Technical learning report  
**Date:** 2026-04-26  
**Publication status:** Draft  
**Affected system type:** Self-hosted containerized mail environment  
**Disclosure level:** Sanitized

---

## Summary

A self-hosted containerized mail environment experienced an inbound email disruption. The mail services were running, and local service checks succeeded, but externally delivered messages did not reliably reach the mailbox layer.

The cause was an interaction between host-level network policy, container networking, and mail-stack isolation behavior. The mail application itself was not the primary failure point. The resolution required correcting the host-to-container traffic path and then validating the complete external SMTP flow.

---

## Environment

The affected environment used:

- a self-hosted mail stack
- containerized service deployment
- host-level network filtering
- container bridge networking
- SMTP inbound delivery
- mailbox delivery through an internal mail delivery component
- external DNS MX records pointing to the self-hosted mail system

This public report intentionally avoids real hostnames, domains, IP addresses, internal paths, private network ranges, organization-specific security architecture, credentials, secrets, tokens, or personal data.

---

## Symptoms

The main symptoms were:

- inbound test emails did not arrive
- containers appeared healthy
- local connectivity checks succeeded
- external delivery still failed
- packet-level diagnostics showed that local health did not equal public reachability

The most important operational lesson was that a service can appear healthy internally while the external delivery path is still broken.

---

## Root Cause

The root cause was a rule-ordering and traffic-path conflict between host-level network filtering and container network isolation.

Externally initiated SMTP traffic reached the host, but the forwarding path into the containerized mail stack was interrupted by network policy ordering. Because of this, the mail service was healthy locally, yet external SMTP traffic could not consistently complete the full path to message delivery.

A separate custom blocking rule contributed to instability because it interfered with the expected order of mail-stack-managed network rules.

---

## Resolution

The incident was resolved by:

1. confirming that the mail containers were running
2. confirming that local service checks were successful
3. comparing local checks with external packet-level behavior
4. identifying the host-to-container traffic path as the failure point
5. adjusting network policy order so required public mail traffic reached the containerized mail service
6. validating external SMTP delivery end-to-end
7. adding a persistence mechanism so the corrected behavior survives host startup

---

## Verification

After remediation, the following signals were observed:

- external connection setup completed
- SMTP banner was returned
- encrypted SMTP negotiation succeeded
- the mail application accepted the message
- final mailbox delivery completed successfully

A successful mail log signal was equivalent to:

```text
status=sent ... Saved
```

---

## Lessons Learned

1. Container health checks are not enough for internet-facing mail systems.
2. Local port checks can pass while public inbound delivery is still broken.
3. Host-level filtering and container networking must be treated as a combined delivery path.
4. Production mail hosts should avoid ad-hoc top-level network changes.
5. External monitoring should validate real SMTP reachability, not only process health.
6. Network rule ownership and change governance are essential for reliable self-hosted mail.

---

## Prevention Measures

Recommended follow-up measures:

- add external SMTP reachability monitoring
- monitor for mail-stack network rule warnings
- document which component owns which network controls
- maintain an inventory of domains and MX targets
- validate the external SMTP path after container or host network changes
- consider a periodic consistency check for expected network policy order

---

## Public Disclosure Notes

This report is sanitized. It does not include:

- real internal hostnames
- real domains
- real IP addresses
- internal filesystem paths
- private network ranges
- organization-specific security architecture
- credentials, passwords, tokens, secrets, or private keys
- personal data

The goal is to share operational lessons from a self-hosted containerized mail incident without exposing sensitive infrastructure details.
