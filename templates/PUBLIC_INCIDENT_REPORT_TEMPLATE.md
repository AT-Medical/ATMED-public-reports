# PUBLIC_INCIDENT_REPORT_TEMPLATE.md

> Template for sanitized public incident and troubleshooting reports.

---
title: "[Public incident title]"
date: "YYYY-MM-DD"
category: "[mail | docker | cloudflare | traefik | linux | monitoring | other]"
tags:
  - "[tag]"
source_internal_report: "[private reference only, do not expose confidential URL if unsafe]"
sanitation_status: "draft"
approval_status: "not-approved"
public_release: false
reddit:
  draft_available: true
  approved: false
  suggested_subreddits: []
  selected_subreddit: null
  posted_url: null
---

# [Public incident title]

## Summary

Briefly describe the problem and the solution in general terms.

## Environment

Describe the generic technical environment:

- operating system family/version if relevant
- software/service involved
- container/runtime environment
- relevant network or proxy layer

Do not include real organization-specific hostnames, IP addresses, domains, usernames, tokens, or private file paths.

## Problem

Describe the issue and symptoms.

## Current System State / Observed Behavior

Describe how the system behaved at the time of the incident.

## Root Cause

Explain the actual underlying cause.

## Fix / Resolution

Describe the steps used to resolve the issue in a safe, generalizable way.

## Verification

Explain how the fix was verified.

## Prevention / Hardening

Describe what was changed or recommended to prevent recurrence.

## Lessons Learned

List reusable insights.

## Disclaimer

This report is a sanitized technical troubleshooting note. Adapt commands and configuration to your own environment. Do not copy firewall, mail, or security rules blindly without understanding their impact.
