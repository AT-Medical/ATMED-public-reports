---
title: "Docker container healthy, Traefik route still returned 404 after recreate"
source_public_report: "incident-reports/infrastructure/2026-06-09_container-route-404-after-recreate.md"
category: "infrastructure"
tags:
  - "traefik"
  - "docker"
  - "reverse-proxy"
  - "selfhosted"
hashtags:
  - "#traefik"
  - "#docker"
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
    - "r/Traefik"
    - "r/selfhosted"
    - "r/docker"
    - "r/sysadmin"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Draft

## Suggested title

Docker container healthy, Traefik route still returned 404 after recreate

## Suggested subreddits and rationale

- `r/Traefik`: Directly relevant to Docker provider routing state.
- `r/selfhosted`: Common pattern in self-hosted stacks.
- `r/docker`: Relates to Docker Compose service/container naming and container recreation.
- `r/sysadmin`: Useful operational lesson about service health vs externally routed health.

## Post body

A small but useful operational reminder: after recreating a Docker Compose service behind Traefik, the container can be healthy and the labels can still look correct, while the externally routed endpoint returns:

```text
404 page not found
```

In this case:

- the Compose service name and container name were different
- after using the correct Compose service name, the container recreated successfully
- the container was healthy
- Traefik labels were present and correct
- the route still returned 404
- restarting Traefik restored the route immediately

The practical takeaway was to treat reverse-proxy route health as a separate verification step after container recreation. Container health alone is not enough.

A simple post-change checklist:

```text
1. Verify container is healthy
2. Verify labels/routing config
3. Verify external route/API endpoint
4. Restart/reload proxy if Docker provider state did not update cleanly
```

Nothing dramatic, but exactly the kind of thing that eats time when you assume “healthy container” means “working route.”

---

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de
