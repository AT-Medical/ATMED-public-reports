---
title: "Authentik admin recovery: akadmin missing and createsuperuser no longer suitable"
source_public_report: "incident-reports/infrastructure/2026-06-03_authentik-admin-bootstrap-recovery.md"
category: "infrastructure"
tags:
  - "authentik"
  - "docker"
  - "selfhosted"
  - "identity-provider"
  - "incident-report"
hashtags:
  - "#authentik"
  - "#selfhosted"
  - "#docker"
  - "#sysadmin"
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
    - "r/authentik"
    - "r/selfhosted"
    - "r/sysadmin"
    - "r/docker"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Draft Reddit Post

## Suggested title

Authentik admin recovery: akadmin missing and createsuperuser no longer suitable

## Suggested subreddits and rationale

- `r/authentik`: most directly relevant to Authentik operators.
- `r/selfhosted`: useful for self-hosted identity provider deployments.
- `r/sysadmin`: relevant because the issue concerns admin recovery, runbooks, and operational hardening.
- `r/docker`: relevant because the deployment used Docker Compose and container recreation.

## Post body

I ran into an Authentik admin recovery situation that may be useful for others running self-hosted Authentik behind a reverse proxy with Docker Compose.

The short version: the expected legacy/default admin user was not present, and the older `createsuperuser` workflow was not the correct recovery path for the deployed Authentik version.

Relevant error signatures:

```text
CommandError: user 'akadmin' does not exist
manage.py createsuperuser: error: unrecognized arguments: --username --email
RuntimeError: ak createsuperuser should not be used. Instead, use ak create_admin_group
Environment variable not found, using fallback: POSTGRES_PASSWORD
```

What worked was the following general pattern:

1. Use Authentik shell to create or update the intended admin user.
2. Use the supported admin-group workflow to grant admin rights.
3. Reset the admin password through the Authentik CLI.
4. Confirm that MFA is still active.
5. Recreate the affected containers.
6. Verify container health and the public login endpoint.
7. Write a recovery runbook and checksum it.

The main lesson: do not rely on default-user assumptions for identity-provider recovery. Test the admin recovery path after version changes and keep a version-aware break-glass runbook.

Hardening steps I would recommend after recovery:

- Add WebAuthn/passkeys as an additional admin factor.
- Keep secrets out of runbooks and repositories.
- Store recovery material in an offline break-glass vault.
- Rotate secrets that were exposed or temporarily backed up during troubleshooting.
- Configure audit notifications for privileged identity events.

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de
