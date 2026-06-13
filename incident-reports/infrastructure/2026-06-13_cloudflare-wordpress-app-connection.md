---
title: "Troubleshooting self-hosted WordPress app connectivity behind a reverse proxy"
date: "2026-06-13"
category: "infrastructure"
classification: "public"
status: "draft"
redaction: "public-sanitized"
---

# Troubleshooting self-hosted WordPress app connectivity behind a reverse proxy

## Summary

A self-hosted WordPress site could not be added to the WordPress mobile app. The issue was caused by a combination of security plugin settings, canonical-domain redirects, reverse-proxy HTTPS detection and an additional login-hiding plugin. After fixing these layers, the WordPress app connected successfully.

## Environment

- Self-hosted WordPress.
- Docker-based deployment.
- Reverse proxy with TLS termination.
- External DNS/proxy/security layer.
- WordPress security plugin.
- Login-hiding plugin.

Specific hostnames, internal paths, IPs and organization details are intentionally omitted.

## Problem

The WordPress app either reported that application passwords might be disabled, hit redirect loops, or displayed a WordPress error page instead of completing site detection.

## Observed Behavior

Common symptoms included:

- Application Passwords not available.
- Excessive redirects between canonical host variants.
- API requests interrupted by a browser-oriented challenge.
- XML-RPC behaving unexpectedly until tested with the correct HTTP method.
- Login-hiding behavior returning error pages for paths the app expected.

## Root Cause

The root cause was a multi-layer configuration mismatch:

1. Application Passwords were disabled by a security plugin.
2. Apex and `www` canonical redirects contradicted each other.
3. WordPress did not consistently detect HTTPS behind the reverse proxy.
4. API-style traffic was treated like browser traffic by the external protection layer.
5. A login-hiding plugin interfered with the WordPress app’s discovery flow.

## Fix / Resolution

The site was restored by:

- Enabling Application Password support.
- Choosing one canonical domain.
- Adding correct HTTPS detection for reverse-proxy headers.
- Allowing WordPress app API endpoints to complete without a browser challenge.
- Disabling the login-hiding plugin for app compatibility.

## Verification

Verification should include:

- REST API endpoint returns HTTP 200.
- XML-RPC returns HTTP 405 for HEAD/GET and HTTP 200 for a valid POST request.
- No redirect loop between apex and `www`.
- WordPress app can add the self-hosted site.

## Prevention / Hardening

- Treat mobile app compatibility as a requirement when adding login-hiding plugins.
- Document reverse-proxy HTTPS settings.
- Keep API endpoint protection rules explicit and minimal.
- Test XML-RPC with POST, not only HEAD/GET.
- Keep WP-CLI access method documented for containerized deployments.

## Lessons Learned

The WordPress app depends on several standard WordPress behaviors. Security layers can break it even when the public website works normally. Troubleshooting should check WordPress settings, canonical domain handling, reverse-proxy headers, API endpoint behavior and login-hiding plugins in that order.

## Disclaimer

This public report is a sanitized technical summary. It does not include hostnames, IP addresses, private infrastructure paths, access tokens, credentials or customer data.
