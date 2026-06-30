---
title: "Paperless-ngx beta image label and application version display mismatch"
source_public_report: "incident-reports/infrastructure/2026-05-12_paperless-beta-image-version-mismatch.md"
category: "infrastructure"
tags: ["paperless-ngx", "docker", "selfhosted"]
hashtags: ["#paperlessngx", "#docker", "#selfhosted"]
mandatory_disclaimer: true
organization:
  name: "AT Medical GmbH"
  location: "Heidelberg"
  website: "https://www.at-medical.de"
  contact: "Support@at-medical.de"
reddit:
  status: "draft"
  approved: false
  suggested_subreddits: ["r/selfhosted", "r/sysadmin", "r/docker"]
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Draft

During a controlled test of a Paperless-ngx beta container, Docker image labels showed the expected beta image, but the application UI and runtime log still displayed an older stable version string. The stack itself was healthy and migrations were current.

Practical lesson: when testing pre-release images, compare multiple version sources instead of relying only on the UI. Image labels, migration state, health checks, and logs can tell different parts of the story.

Suggested subreddits: `r/selfhosted`, `r/sysadmin`, `r/docker`.

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
