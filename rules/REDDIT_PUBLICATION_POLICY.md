# REDDIT_PUBLICATION_POLICY.md — Reddit Drafts and Publication

## Purpose

This policy defines how sanitized public reports may be prepared and later published to Reddit.

## Mandatory Transparency Disclaimer

Every Reddit post generated from an ATMED public incident report must include the following disclaimer either at the beginning or as a footer.

Preferred English version:

```text
Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg
Website: https://www.at-medical.de
Contact: Support@at-medical.de
```

German source version:

```text
Transparenzhinweis: Hierbei handelt es sich um einen automatisch generierten Incident Report auf Grundlage einer realen Fehlerbehebung mithilfe von ChatGPT. Die Zusammenfassung wurde über GitHub durch einen automatisierten Worker erstellt und erst nach Freigabe durch die Abteilung IT und Sicherheit der AT Medical GmbH, Heidelberg veröffentlicht.

Organisation: AT Medical GmbH, Heidelberg
Webseite: https://www.at-medical.de
Kontakt: Support@at-medical.de
```

The Reddit worker must refuse publication if the disclaimer, website, or contact address is missing.

## Required Contact Information

Every Reddit post must include:

```text
Website: https://www.at-medical.de
Contact: Support@at-medical.de
```

The link to the full public GitHub report should remain in the report body itself.

## Recommendation

Do **not** store a Reddit username and password in GitHub Secrets.

Use one of the following safer patterns:

1. Reddit OAuth application credentials stored as secrets.
2. A VPS worker using credentials stored in a private vault.
3. Manual posting from a prepared Reddit draft.

## Why Not Username/Password?

Storing a Reddit password in GitHub Secrets is not recommended because:

- it grants broad account access,
- password-based automation is fragile,
- two-factor authentication may break flows,
- account compromise risk is higher,
- OAuth/API credentials can be scoped and rotated more safely.

## Preferred Architecture

```text
ATMED-public-reports Reddit draft
  -> approval_status / reddit.approved = true
  -> selected_subreddit set
  -> disclaimer validation
  -> GitHub Action or VPS worker reads draft
  -> Reddit API OAuth post
  -> posted URL written back to draft/report
  -> NTFY notification
```

## Recommended Secrets

If using GitHub Actions for posting:

```text
REDDIT_CLIENT_ID
REDDIT_CLIENT_SECRET
REDDIT_REFRESH_TOKEN
REDDIT_USER_AGENT
```

Optional:

```text
REDDIT_USERNAME
```

Do not store:

```text
REDDIT_PASSWORD
```

unless there is no alternative and the risk has been explicitly accepted. Prefer OAuth refresh tokens.

## Safer Long-Term Option

Use a VPS worker with credentials stored outside GitHub, for example in:

- Vaultwarden
- encrypted `.env` outside the repository
- systemd environment file with restricted permissions
- future ATMED vault/secrets service

## Publication Gate

A Reddit draft must not be posted unless:

```yaml
reddit:
  approved: true
  selected_subreddit: "r/..."
mandatory_disclaimer: true
organization:
  website: "https://www.at-medical.de"
  contact: "Support@at-medical.de"
```

and the sanitization checklist is complete.

## Draft Requirements

Every Reddit draft should include:

```yaml
hashtags:
  - "#Mailcow"
  - "#Docker"
  - "#iptables"
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
    - "r/mailcow"
    - "r/selfhosted"
  selected_subreddit: null
  posted_url: null
```

## Posting Behavior

When approved, a worker may:

1. validate sanitization metadata,
2. validate mandatory disclaimer is present,
3. validate website and contact address are present,
4. validate selected subreddit,
5. submit the post via Reddit API,
6. store the posted URL in the draft metadata or a publication log,
7. notify Andreas via NTFY.

## Final Rule

Prepare automatically. Publish only after explicit approval. Never publish without the mandatory transparency disclaimer, website, and contact address.
