---
title: "Troubleshooting IPv6, DNS AAAA parity, Linux firewall ICMPv6, and Ubuntu MOTD cache issues"
date: "2026-06-19"
category: "infrastructure"
tags:
  - "ipv6"
  - "cloudflare"
  - "dns"
  - "linux"
  - "firewall"
  - "nftables"
  - "motd"
  - "reverse-proxy"
source_internal_report: "internal reference only"
sanitation_status: "draft"
approval_status: "not-approved"
public_release: false
reddit:
  draft_available: true
  approved: false
  suggested_subreddits:
    - "r/sysadmin"
    - "r/linuxadmin"
    - "r/cloudflare"
    - "r/selfhosted"
  selected_subreddit: null
  posted_url: null
---

# Troubleshooting IPv6, DNS AAAA parity, Linux firewall ICMPv6, and Ubuntu MOTD cache issues

## Summary

A multi-part infrastructure troubleshooting session resolved several issues that appeared related but had separate root causes: a web service behind a reverse proxy was not reliably reachable, Ubuntu login messages reported failed release metadata checks, several servers lacked working IPv6 connectivity, one host blocked essential ICMPv6 traffic through its firewall, and DNS-only hostnames had A records without corresponding AAAA records.

The remediation involved separating proxy/DNS issues from application health, clearing stale Ubuntu MOTD cache state, enabling and persisting IPv6 routes, allowing essential ICMPv6 in the firewall, and auditing DNS records for IPv6 parity.

## Environment

Generic environment:

- Linux servers running Ubuntu LTS versions
- Dockerized web services behind a reverse proxy
- Cloudflare-managed DNS zones
- Mixed Cloudflare-proxied and DNS-only hostnames
- UFW and nftables firewall layers
- IPv4 and IPv6 public server addressing

No real hostnames, IP addresses, internal paths, tokens, or private infrastructure details are included in this public draft.

## Problem

Several symptoms appeared during the same operational window:

- A web application behind a reverse proxy was reachable internally but not reliably from the public internet.
- Cloudflare or DNS state affected public access and challenge behavior.
- Ubuntu login banners showed release metadata connectivity errors.
- `curl -6` checks failed on several servers even though IPv4 worked.
- A DNS audit found DNS-only A records without AAAA records.

## Current System State / Observed Behavior

The application layer was healthy internally. The public access issue was instead caused by DNS/proxy configuration and edge behavior. Separately, IPv6 was inconsistently configured across hosts. In one case, a server had an IPv6 address and route but could not reach the IPv6 gateway because firewall rules prevented necessary ICMPv6 neighbor discovery traffic.

Ubuntu release metadata warnings persisted even after connectivity was available because the message was stored in a local cached release-upgrader state file.

## Root Cause

The incident had multiple root causes:

1. **Reverse proxy / DNS issue:** The public hostname configuration did not match the actual intended routing and Cloudflare state.
2. **Large reverse-proxy host/SAN grouping:** Too many domains in one proxy/TLS grouping can make a single misconfigured or unavailable DNS zone affect unrelated services.
3. **Ubuntu MOTD cache:** A stale release-upgrader cache file contained an old error message and continued to display it during login.
4. **IPv6 host configuration:** Some systems had no active global IPv6 address or lacked a stable source-specific default route.
5. **Firewall / ICMPv6:** One host blocked essential ICMPv6, which broke IPv6 neighbor discovery and gateway reachability.
6. **DNS AAAA parity:** DNS-only A records were not consistently paired with AAAA records.

## Fix / Resolution

The general remediation pattern was:

1. Test the application internally before changing application code.
2. Verify proxy labels/routes, certificates, and DNS records separately.
3. For Ubuntu login warnings, inspect the release-upgrader cache file before disabling MOTD scripts.
4. Enable IPv6 using assigned addresses from the provider metadata or provisioning configuration.
5. Add a default IPv6 route via the provider gateway with an explicit source address.
6. Ensure firewall rules allow essential ICMPv6 and established/related traffic.
7. Audit DNS zones for DNS-only A records without AAAA records and add AAAA records only where the IPv6 target is known.

## Verification

Verification steps included:

- `curl -6` to a known external HTTPS endpoint from every server.
- IPv4 sanity checks after IPv6 changes.
- Gateway ping checks over IPv6.
- Direct execution of the Ubuntu MOTD release-upgrade script to confirm the warning was gone.
- DNS API audit to confirm that DNS-only A records had corresponding AAAA records where appropriate.

## Prevention / Hardening

Recommended long-term measures:

- Include IPv6 checks in routine server health checks.
- Audit DNS-only A/AAAA parity periodically.
- Treat proxied and DNS-only Cloudflare records differently.
- Avoid very large reverse-proxy host/SAN groupings.
- Validate DNS zone availability before including a domain in automated DNS-01 certificate workflows.
- Never block ICMPv6 broadly; IPv6 depends on it for neighbor discovery and path MTU behavior.
- Store operational API tokens only in approved secret stores and never in documentation or reports.

## Lessons Learned

- IPv6 can appear configured while still being broken if ICMPv6 is blocked.
- Login warnings may be stale cache state rather than current network state.
- DNS-only records require explicit AAAA records for IPv6 client reachability.
- Cloudflare-proxied records and DNS-only records must not be evaluated with the same assumptions.
- Central orchestration is safer and more repeatable than manual per-host remediation.

## Disclaimer

This report is a sanitized technical troubleshooting note. Adapt commands and configuration to your own environment. Do not copy firewall, mail, DNS, or security rules blindly without understanding their impact.
