# REDDIT_DRAFT_TEMPLATE.md

> Template for Reddit-ready sanitized distribution posts.

---
title: "[Reddit title]"
source_public_report: "[path or URL]"
category: "[mail | docker | cloudflare | traefik | linux | monitoring | other]"
tags:
  - "[tag]"
hashtags:
  - "#[Tag]"
mandatory_disclaimer: true
reddit:
  status: "draft"
  approved: false
  suggested_subreddits:
    - "r/..."
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Suggested Reddit Title

[Search-friendly title]

## Suggested Subreddits

| Rank | Subreddit | Reason |
|---:|---|---|
| 1 | r/... | |
| 2 | r/... | |
| 3 | r/... | |

## Mandatory Transparency Disclaimer

This text must appear either at the beginning or as a footer of every Reddit post:

```text
Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.
```

German source version:

```text
Transparenzhinweis: Hierbei handelt es sich um einen automatisch generierten Incident Report auf Grundlage einer realen Fehlerbehebung mithilfe von ChatGPT. Die Zusammenfassung wurde über GitHub durch einen automatisierten Worker erstellt und erst nach Freigabe durch die Abteilung IT und Sicherheit der AT Medical GmbH, Heidelberg veröffentlicht.
```

## Draft Post

[Insert mandatory transparency disclaimer here, either at the beginning or as a footer.]

I ran into a technical issue that may be useful for others troubleshooting a similar setup.

**Environment**

- Generic environment summary
- No real hostnames, domains, IPs, usernames, or private paths

**Problem**

Short description of the issue and symptoms.

**Root cause**

Short explanation of what caused it.

**Fix**

Short, safe, generalized fix description.

**Verification**

How the fix was verified.

**Prevention**

What to change to avoid this recurring.

Full write-up: [link to public report]

## Review Checklist

- [ ] Mandatory transparency disclaimer included
- [ ] No real IPs
- [ ] No real hostnames/domains unless intentionally public
- [ ] No secrets/tokens
- [ ] No customer/patient/person data
- [ ] No internal paths
- [ ] No sensitive security architecture details
- [ ] Subreddit rules checked
- [ ] Approved by Andreas
- [ ] Approved by IT and Security
