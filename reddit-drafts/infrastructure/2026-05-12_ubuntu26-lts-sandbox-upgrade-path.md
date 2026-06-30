---
title: "Ubuntu 24.04 LTS to 26.04 LTS: development upgrade path blocked, ISO lab install chosen"
source_public_report: "incident-reports/infrastructure/2026-05-12_ubuntu-lts-development-upgrade-blocked.md"
category: "infrastructure"
tags: ["ubuntu", "lts", "server", "upgrade"]
hashtags: ["#ubuntu", "#linux", "#sysadmin"]
mandatory_disclaimer: true
organization:
  name: "AT Medical GmbH"
  location: "Heidelberg"
  website: "https://www.at-medical.de"
  contact: "Support@at-medical.de"
reddit:
  status: "draft"
  approved: false
  suggested_subreddits: ["r/Ubuntu", "r/sysadmin", "r/linuxadmin", "r/selfhosted"]
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Draft

In a non-production lab, an Ubuntu 24.04 LTS server was prepared for early Ubuntu 26.04 LTS testing. The release upgrader did not offer the regular LTS upgrade yet, and the development mode path returned that upgrades to the development release are only available from the latest supported release.

The practical conclusion for this lab: use the provider's Ubuntu 26.04 server ISO and perform a clean install instead of forcing a multi-step upgrade path.

Suggested subreddits: `r/Ubuntu`, `r/sysadmin`, `r/linuxadmin`, `r/selfhosted`.

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
