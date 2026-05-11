---
title: "Nextcloud 33 desktop client fails after successful Login-v2 auth with QueryBuilder::execute() error"
source_public_report: "incident-reports/infrastructure/2026-05-11_nextcloud-querybuilder-client-error.md"
category: "infrastructure"
tags:
  - "nextcloud"
  - "docker"
  - "selfhosted"
  - "traefik"
  - "oauth"
hashtags:
  - "#nextcloud"
  - "#selfhosted"
  - "#docker"
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
    - "r/NextCloud"
    - "r/selfhosted"
    - "r/docker"
    - "r/sysadmin"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

We recently ran into an interesting Nextcloud 33 desktop client issue.

Symptoms:

- Browser-based Login-v2 / OAuth flow completed successfully
- `/status.php` returned healthy status
- Desktop client still failed immediately afterward
- misleading SSL client-certificate prompts appeared

Main backend error:

```text
Call to undefined method OCP\\DB\\QueryBuilder\\QueryBuilder::execute()
```

Root cause appears to be an outdated/incompatible Nextcloud app still using the deprecated QueryBuilder API after upgrading to NC33.

Useful commands:

```bash
php occ app:list
php occ app:update --all
php occ log:watch
```

We also checked:

- reverse proxy TLS policies
- accidental mTLS enforcement
- SSO/OAuth plugins

Interesting takeaway:

The TLS/client-certificate popup initially looked like a reverse proxy or Cloudflare issue, but it seems the desktop client was surfacing secondary authentication/handshake failures triggered by the backend exception.

Has anyone else seen this exact behavior after upgrading to NC33?

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
