---
title: "Debugging a mixed IPv6/DNS/Cloudflare/Linux firewall issue: app was fine, infra was not"
source_public_report: "incident-reports/infrastructure/2026-06-19_ipv6-cloudflare-dns-linux-firewall-motd.md"
category: "infrastructure"
tags:
  - "ipv6"
  - "cloudflare"
  - "dns"
  - "nftables"
  - "linux"
  - "motd"
hashtags:
  - "#IPv6"
  - "#Cloudflare"
  - "#DNS"
  - "#Linux"
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
    - "r/sysadmin"
    - "r/linuxadmin"
    - "r/cloudflare"
    - "r/selfhosted"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

Debugging a mixed IPv6/DNS/Cloudflare/Linux firewall issue: app was fine, infra was not

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/sysadmin | Broad infrastructure troubleshooting with DNS, IPv6, reverse proxy and Linux server operations. |
| 2 | r/linuxadmin | Linux-specific IPv6, nftables, UFW and MOTD cache behavior. |
| 3 | r/cloudflare | DNS-only vs proxied Cloudflare behavior and AAAA parity auditing. |
| 4 | r/selfhosted | Self-hosted services behind reverse proxies with mixed DNS/firewall issues. |

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

I ran into a multi-layer infrastructure issue that may be useful to others debugging similar setups.

**Environment**

- Ubuntu LTS servers
- Dockerized application behind a reverse proxy
- Cloudflare DNS with a mix of proxied and DNS-only records
- IPv4 + IPv6 public addressing
- UFW and nftables firewall layers

**Problem**

A web application looked broken publicly, but the app itself was fine internally. At the same time, several servers had IPv6 failures, Ubuntu login banners showed release metadata connection errors, and DNS-only records lacked matching AAAA records.

**Root cause**

It turned out to be multiple separate infrastructure issues:

- DNS/proxy configuration affected public access.
- A large reverse-proxy host/SAN grouping made unrelated domain issues harder to isolate.
- Ubuntu MOTD was showing stale cached release-upgrader errors.
- Some servers did not have working global IPv6 configuration.
- One host blocked essential ICMPv6/neighbor discovery through nftables.
- DNS-only A records did not consistently have AAAA parity.

**Fix**

The general fix was:

1. Confirm the application was healthy internally before touching app code.
2. Separate reverse proxy, DNS and Cloudflare behavior during testing.
3. Clear stale Ubuntu release-upgrader cache only after confirming the error signature.
4. Configure IPv6 address + source-specific default route from provider metadata.
5. Allow essential ICMPv6 and established/related traffic in the firewall.
6. Audit DNS-only A records and add AAAA records only for known IPv6-capable targets.

**Verification**

- `curl -6` from all affected hosts returned HTTP 200 to a known external endpoint.
- IPv4 remained working.
- DNS audit no longer showed DNS-only A records without AAAA records.
- The Ubuntu MOTD release-upgrade warning disappeared.

**Prevention**

- Add IPv6 checks to regular server health checks.
- Audit Cloudflare DNS-only A/AAAA parity periodically.
- Avoid huge reverse-proxy host/SAN groups.
- Do not broadly block ICMPv6.
- Treat Cloudflare-proxied and DNS-only records as different operational cases.

Full write-up: `incident-reports/infrastructure/2026-06-19_ipv6-cloudflare-dns-linux-firewall-motd.md`

## Footer / Contact

```text
Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
```

## Review Checklist

- [x] Mandatory transparency disclaimer included
- [x] Website and contact address included
- [x] Link/reference to full public GitHub report included
- [x] No real IPs
- [x] No real hostnames/domains except organization contact block required by template
- [x] No secrets/tokens
- [x] No customer/patient/person data
- [x] No internal paths
- [x] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
