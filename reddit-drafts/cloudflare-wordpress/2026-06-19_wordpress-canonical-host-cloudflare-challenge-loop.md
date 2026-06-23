---
title: "WordPress behind Cloudflare: canonical host switch to www caused browser challenge/redirect-loop symptoms"
source_public_report: "incident-reports/cloudflare-wordpress/2026-06-19_wordpress-canonical-host-cloudflare-challenge-loop.md"
category: "cloudflare-wordpress"
tags:
  - "wordpress"
  - "cloudflare"
  - "redirect-loop"
  - "canonical-host"
  - "waf"
hashtags:
  - "#WordPress"
  - "#Cloudflare"
  - "#sysadmin"
  - "#selfhosted"
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
    - "r/cloudflare"
    - "r/sysadmin"
    - "r/selfhosted"
    - "r/Wordpress"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Reddit Draft

## Suggested title

WordPress behind Cloudflare: canonical host switch to www caused browser challenge/redirect-loop symptoms

## Suggested subreddits and rationale

- **r/cloudflare** — best fit because the user-visible symptoms involved Cloudflare challenge state, proxied DNS, Browser Check/WAF behavior, and `cf-mitigated: challenge` indicators.
- **r/sysadmin** — relevant as a production change-management and troubleshooting lesson involving DNS/CDN/WAF/reverse proxy behavior.
- **r/selfhosted** — relevant for operators running WordPress behind Cloudflare and a self-managed reverse proxy/container stack.
- **r/Wordpress** — relevant because the immediate trigger was changing `siteurl`, `home`, `WP_HOME`, and `WP_SITEURL`.

## Draft body

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de

---

We ran into a useful but painful WordPress + Cloudflare lesson that may help other admins.

A WordPress site behind Cloudflare had a legitimate security hardening task: Wordfence reported that a sensitive configuration file was publicly reachable. We fixed that correctly at the webserver layer, and sensitive files returned `403 Forbidden` afterwards.

The problem started when, during the same maintenance session, the site was also switched from the apex domain to `www` as the WordPress canonical host. The changes included:

```text
siteurl
home
WP_HOME
WP_SITEURL
```

Local origin and reverse-proxy checks looked fine:

```text
apex -> 301 -> www
www  -> 200
sensitive files -> 403
```

But real browsers started failing. Symptoms included:

```text
connection not secure
too many redirects
Cloudflare challenge token in the URL
cf-mitigated: challenge
```

The important part: the origin was not necessarily broken. The problem was the interaction between the new WordPress canonical redirect, existing Cloudflare challenge/HTTPS behavior, and browser-held Cloudflare challenge state.

The fix was not more random Cloudflare tweaking. The actual recovery happened after rolling WordPress back to the previous canonical host and removing the hardcoded `WP_HOME` / `WP_SITEURL` constants. The sensitive-file hardening stayed in place.

## Lessons learned

- Do not combine a security hardening fix with a canonical-host migration in the same live change window.
- A `200 OK` from the origin or local reverse proxy does not prove end-user availability behind Cloudflare.
- Cloudflare-backed WordPress canonical changes should be tested as a full browser-to-edge-to-origin flow.
- `WP_HOME` and `WP_SITEURL` are powerful. Do not hardcode them during live troubleshooting unless you have a rollback ready.
- URLs containing `__cf_chl_tk` are Cloudflare challenge artifacts, not normal test URLs.
- Safari/iOS testing matters. It can expose redirect/challenge problems that terminal checks miss.

## Practical prevention checklist

Before changing a WordPress site from apex to `www` behind Cloudflare:

1. Audit Cloudflare redirects, WAF/custom rules, Browser Check, and HTTPS settings.
2. Prepare the rollback command before changing WordPress.
3. Change one layer at a time.
4. Test apex and `www` from real browsers.
5. Avoid hardcoded `WP_HOME` / `WP_SITEURL` unless strictly necessary.
6. Keep the security fix and the canonical migration as separate changes.

This was a good reminder that CDN/WAF/browser state is part of production, not just the origin server.

## Publication status

Draft only. Not approved for publication.
