---
title: "Restic check and stale lock handling in a self-hosted backup control plane"
date: "2026-05-11"
category: "infrastructure"
classification: "public-redacted"
---

# Public Incident Report: Restic check and stale lock handling

## Summary

A self-hosted backup control plane using restic, rclone, systemd and a web dashboard reported failed backup runs during final stabilization. In the most relevant case, snapshot creation succeeded, but the later repository verification phase failed and was displayed as a generic script error.

## Generalized Environment

- Linux backup server
- Multiple remote Linux endpoints over SSH
- restic backup engine
- rclone transport to S3-compatible object storage
- two independent storage targets
- local metadata database
- web dashboard
- systemd service and timer
- encrypted configuration stored outside the source repository

## Problem

The system mixed backup creation, repository checks and restore tests into one operational workflow. When repository verification returned a non-zero result, the global shell error handling surfaced this as a generic line-number failure instead of a structured check failure.

A separate stale-lock situation also appeared after an interrupted or long-running restic operation.

## Root Cause

The incident was caused by a combination of:

- direct live patching of operational scripts
- fragile shell error handling
- expected verification failures not being handled as structured states
- stale restic locks after interrupted or long-running operations
- dashboard and backend code being changed together during rapid iteration

## Resolution

The remediation approach was:

1. Restore the last syntactically valid script state.
2. Verify endpoint connectivity.
3. Confirm no active backup or restic/rclone process was still running.
4. Remove stale locks only after this confirmation.
5. Adjust the model so verification failures are recorded as check failures.
6. Move ongoing development into version control with issues, runbooks and reviewable changes.

## Verification

Verification included:

- checking timer and service state
- checking endpoint reachability
- confirming snapshot coverage across endpoints and storage targets
- reviewing backup metadata
- running repository checks
- running mini-restore tests where available

## Prevention

Recommended hardening:

- version-control backup logic
- avoid direct live patching
- separate dashboard UI from backup execution
- add Bash and Python checks in CI
- use shell linting
- document stale-lock handling
- perform full restore drills
- keep sensitive configuration outside Git
- use release tags and changelogs

## Lessons Learned

A backup system is not enterprise-ready just because snapshots exist. Restore tests, verification, clear failure states and controlled change management are required.

## Disclaimer

This report is generalized. Organization-specific hostnames, domains, IP addresses, internal paths and sensitive values were removed.