---
title: "Next.js build failed after repeated global CSS dashboard customizations"
source_public_report: "ATMED-public-reports/incident-reports/frontend/2026-06-13_nextjs-css-customization-build-failure.md"
category: "frontend"
tags:
  - "nextjs"
  - "css"
  - "docker"
  - "selfhosted"
hashtags:
  - "#NextJS"
  - "#CSS"
  - "#Docker"
  - "#SelfHosted"
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
    - "r/nextjs"
    - "r/webdev"
    - "r/selfhosted"
    - "r/docker"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

Next.js build failed after repeated global CSS patches during dashboard customization — lessons learned

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/nextjs | Directly relevant because the failure happened during a Next.js/Turbopack build. |
| 2 | r/webdev | Useful general frontend workflow lesson about global CSS customization and component-based UI work. |
| 3 | r/selfhosted | Relevant because the app was self-hosted and rebuilt via Docker. |
| 4 | r/docker | Secondary relevance because the failure surfaced during a Docker image build. |

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

I ran into a frontend customization issue that may be useful for others working on self-hosted Next.js dashboards.

**Environment**

- Next.js / React dashboard
- Global CSS stylesheet
- Docker-based deployment
- Reverse proxy in front of the app

**Problem**

The goal was to make what looked like small UI changes: hide a support button, add a custom logo, and place a footer underneath a fixed bottom navigation bar.

Instead of rebuilding the UI as a component, several global CSS snippets were appended iteratively. The UI still did not behave as intended, and eventually the Docker build failed during the Next.js build stage.

**Build error**

```text
CssSyntaxError: globals.css: Unexpected }
Turbopack build failed
```

**Root cause**

The root cause was not the framework or image format. It was an unsafe customization workflow:

- repeated global CSS append patches,
- old customization remnants not removed,
- selectors targeting fixed-position UI elements indirectly,
- partial cleanup that left a broken CSS structure,
- long shell/SSH transformation commands with fragile quoting.

**Fix**

The safest recovery path was:

1. restore the affected files from source control,
2. reapply only the minimal intended change,
3. run a build before restart,
4. move future branding/layout changes into explicit React components or CSS modules.

**Lesson learned**

If a footer or overlay refuses to move after multiple CSS changes, the problem is often not the value of `bottom`, but the fact that the wrong element is being targeted or older CSS remnants are still active.

For dashboard-style apps with fixed overlays, component-level changes are much safer than repeated global CSS overrides.

Full write-up: [link to public report after approval]

## Review Checklist

- [x] Mandatory transparency disclaimer included
- [x] Website and contact address included
- [ ] Link to full public GitHub report inserted after approval
- [x] No real IPs
- [x] No real hostnames/domains in technical environment section
- [x] No secrets/tokens
- [x] No customer/patient/person data
- [x] No internal paths
- [x] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
