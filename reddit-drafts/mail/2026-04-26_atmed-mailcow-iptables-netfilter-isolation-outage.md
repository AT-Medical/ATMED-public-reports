---
draft_type: "reddit-post"
publication_status: "draft-only"
approved: false
selected_subreddit: null
suggested_subreddits:
  - rank: 1
    subreddit: "r/mailcow"
    rationale: "Most directly relevant to Mailcow operators and self-hosted mail troubleshooting."
  - rank: 2
    subreddit: "r/selfhosted"
    rationale: "Relevant to self-hosted infrastructure, containerized services and operational lessons."
  - rank: 3
    subreddit: "r/sysadmin"
    rationale: "Relevant to production operations, monitoring, incident prevention and mail delivery reliability."
  - rank: 4
    subreddit: "r/docker"
    rationale: "Relevant to container networking and host-to-container reachability issues."
hashtags:
  - "#Mailcow"
  - "#Docker"
  - "#iptables"
  - "#SMTP"
  - "#SelfHosted"
public_report_path: "ATMED-public-reports/incident-reports/mail/2026-04-26_atmed-mailcow-iptables-netfilter-isolation-outage.md"
website: "https://www.at-medical.de"
contact: "Support@at-medical.de"
---

# Reddit Draft – Self-hosted mail inbound failure caused by host/container network rule ordering

**Status:** Draft only  
**Approved:** false  
**Selected subreddit:** null  
**Do not post automatically.**

---

## Suggested title

Self-hosted mail outage: containers were healthy, local SMTP worked, but inbound delivery failed because the host-to-container path was broken

---

## Suggested subreddits

1. **r/mailcow** – best fit for Mailcow operators and similar troubleshooting.
2. **r/selfhosted** – good fit for self-hosted infrastructure lessons.
3. **r/sysadmin** – good fit for monitoring, governance and incident-prevention perspective.
4. **r/docker** – narrower fit, useful if focusing on container networking aspects.

---

## Draft post

I ran into a useful self-hosted mail troubleshooting case that may help others.

The short version: the mail stack itself looked healthy. Containers were running, local SMTP checks passed, and the mailbox service was reachable internally. But external inbound messages still did not arrive.

The key lesson was that **local service health did not prove public mail reachability**.

The actual failure was in the host-to-container traffic path. Host-level network policy, container networking, and mail-stack isolation behavior interacted in a way that prevented external SMTP traffic from completing the full path into the containerized mail service. The mail application was not the primary failure point.

What helped:

- checking container status first, but not stopping there
- comparing local port checks with external packet-level behavior
- validating whether external SMTP traffic reached the host
- checking whether the SMTP session completed end-to-end
- confirming final mailbox delivery in the mail logs
- adding persistence for the corrected network policy behavior

The biggest takeaway for me:

> For self-hosted mail, monitoring should not only check whether the container is running. It should verify real external SMTP reachability and ideally the SMTP banner or a controlled test delivery.

I also learned that ad-hoc top-level firewall changes on a production mail host are a fast way to create weird mail failures. Host firewall rules, container networking and mail-stack-managed network rules need clear ownership and governance.

I wrote a sanitized public incident report here:

`ATMED-public-reports/incident-reports/mail/2026-04-26_atmed-mailcow-iptables-netfilter-isolation-outage.md`

Website: https://www.at-medical.de  
Contact: Support@at-medical.de

---

## Transparency note

This is a sanitized incident write-up from a real self-hosted mail troubleshooting case. It intentionally omits real hostnames, domains, IP addresses, internal paths, internal network ranges, organization-specific security details, credentials and personal data. No Reddit post should be published from this draft without explicit human approval.

---

## Hashtags

#Mailcow #Docker #iptables #SMTP #SelfHosted
