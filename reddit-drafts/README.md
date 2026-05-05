# Reddit Drafts

This directory contains short, sanitized Reddit-ready drafts derived from public incident reports.

## Purpose

Reddit drafts are distribution snippets, not the canonical article. The canonical public article should live under:

```text
incident-reports/<category>/<slug>.md
```

## Publication Rule

Do not post automatically unless the draft contains:

```yaml
reddit:
  approved: true
  selected_subreddit: "..."
```

Manual review is required before posting.

## Suggested Workflow

```text
Public incident report
  -> Reddit draft
  -> suggested subreddit ranking
  -> manual approval
  -> automated posting via Reddit API / VPS Worker
  -> posted URL written back to report/draft
```

## Canonical Template

Use:

```text
templates/REDDIT_DRAFT_TEMPLATE.md
```
