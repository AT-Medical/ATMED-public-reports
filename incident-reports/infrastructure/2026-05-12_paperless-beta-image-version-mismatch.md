---
title: "Paperless-ngx Beta Image Version Mismatch"
date: "2026-05-12"
category: "infrastructure"
classification: "public-sanitized"
tags:
  - "paperless-ngx"
  - "docker"
  - "self-hosted"
  - "beta"
---

# Paperless-ngx Beta Image Version Mismatch

## Summary

During a controlled test of a Paperless-ngx beta container image, Docker image metadata reported the expected beta tag while the application UI and runtime log still displayed the previous stable version string.

## Environment

- Self-hosted Docker deployment
- Paperless-ngx beta image
- PostgreSQL database
- Redis broker
- Reverse proxy in front of the application

## Problem

The container image metadata indicated the beta image, but the application status view and runtime log still reported the older stable version. The application itself remained healthy and reachable.

## Observed Behavior

```text
Docker image label: beta release
Application status/log: older stable version
```

A WebSocket warning was also observed in the UI and was treated as a separate follow-up item.

## Root Cause

The most likely explanation is stale version metadata or a display bug in the beta image. The applied migrations and image labels suggested the intended beta image was active.

## Fix / Resolution

- Pin the container image to a concrete tag instead of a floating tag.
- Recreate the stack after pulling the image.
- Verify database migrations and health checks.
- Check reverse proxy origin and host settings.
- Do not import real document archives until OCR, search, backup, and UI status checks pass.

## Verification

- Container was healthy.
- Database migrations were current.
- Broker and search functions appeared operational.
- Version display mismatch remained an upstream observation.

## Prevention / Hardening

- Avoid `latest` for critical DMS deployments.
- Use harmless test documents after major upgrades.
- Keep a post-upgrade backup.
- Track follow-up release candidates or final releases.

## Disclaimer

This is a sanitized technical report. Organization-specific infrastructure details have been removed.
