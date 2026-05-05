# REDDIT_PUBLICATION_POLICY.md — Reddit Drafts and Publication

## Purpose

This policy defines how sanitized public reports may be prepared and later published to Reddit.

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
```

and the sanitization checklist is complete.

## Draft Requirements

Every Reddit draft should include:

```yaml
hashtags:
  - "#Mailcow"
  - "#Docker"
  - "#iptables"

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
2. validate selected subreddit,
3. submit the post via Reddit API,
4. store the posted URL in the draft metadata or a publication log,
5. notify Andreas via NTFY.

## Final Rule

Prepare automatically. Publish only after explicit approval.
