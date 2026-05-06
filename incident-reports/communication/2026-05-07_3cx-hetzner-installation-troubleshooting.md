---
title: "3CX Manual VPS Installation: Configuration File Workflow and Temporary Port 5015 Troubleshooting"
date: "2026-05-07"
category: "communication"
classification: "public-redacted"
sensitivity: "low"
status: "draft"
review_required: true
tags:
  - "3cx"
  - "vps"
  - "pbx"
  - "voip"
  - "debian"
  - "firewall"
  - "selfhosted"
---

# 3CX Manual VPS Installation: Configuration File Workflow and Temporary Port 5015 Troubleshooting

## Summary

During a manual 3CX PBX installation on a Debian-based VPS, the initial setup was blocked by two common issues: the temporary local web configuration port was not reachable from the browser, and the 3CX portal workflow initially led toward hosted or guided cloud deployment options rather than a manual Linux installation workflow.

The installation was successfully completed by temporarily allowing the setup port, generating the correct configuration file through the manual Linux/on-premise path, uploading it to the local 3CX web configuration page, and verifying that the PBX management console was running on the VPS.

## Environment

- 3CX PBX version 20.x.
- Debian-based VPS.
- Manual ISO / Debian installation path.
- Browser-based 3CX web configuration tool.
- Temporary setup port: TCP 5015.

All organization-specific names, domains, IP addresses and licensing details have been removed.

## Problem

The setup process encountered three practical issues:

1. The browser could not reach the local 3CX configuration tool on port 5015.
2. The 3CX portal workflow initially suggested hosted or guided cloud deployment paths that were not suitable for the existing VPS.
3. A hosted PBX instance could accidentally be created if the wrong portal path is selected.

## Observed Behavior

- Browser error: connection timeout when opening the temporary configuration URL on port 5015.
- Later, connection refused on port 5015 after setup was completed; this is expected once the temporary setup service has stopped.
- The 3CX portal showed hosted/cloud deployment options rather than a generic manual VPS provider workflow.

## Root Cause

### Port 5015 not reachable

The local 3CX web configuration service was running, but the external firewall did not allow inbound TCP traffic on port 5015 during the setup phase.

### Wrong portal path

The guided cloud deployment path is intended for supported cloud providers. For a generic VPS or a manual Linux install, the correct path is to generate/download a configuration setup file and upload it to the local 3CX web configuration tool.

## Fix / Resolution

1. Temporarily allow inbound TCP 5015 from the administrator's network or, during a controlled setup window, from the required source range.
2. Open the local 3CX web configuration URL in the browser using HTTP and port 5015.
3. In the 3CX portal, use the manual Linux / on-premise workflow rather than a hosted deployment.
4. Download or generate the configuration setup file.
5. Upload the configuration setup file to the local 3CX web configuration page.
6. Complete the setup and open the 3CX Admin Console.
7. Remove the temporary firewall rule for port 5015 after setup is complete.
8. Run the 3CX firewall checker and configure the final required PBX ports.

## Verification

Successful verification included:

- Admin console reachable through the final PBX FQDN.
- PBX displayed local/self-managed installation details.
- The VPS reacted to administrative OS actions initiated from the 3CX Admin Console.
- SSH access to the VPS remained available.

## Prevention / Hardening

- Document that port 5015 is temporary and should not remain open after setup.
- Document the difference between hosted, guided cloud deployment, and manual Linux/on-premise installation paths.
- Run the 3CX firewall checker before production use.
- Configure remote backups immediately after installation.
- Keep PBX host access, 3CX portal access, and PBX administrator access conceptually separate.

## Lessons Learned

- A generic VPS install may require the manual Linux/on-premise configuration file workflow rather than a provider-specific cloud deployment assistant.
- A temporary setup port can make an otherwise successful installation appear broken if blocked by a firewall.
- Hosted PBX provisioning and self-managed/local PBX setup are easy to confuse during the portal workflow.

## Disclaimer

This report is a sanitized technical summary based on one real troubleshooting process. It is not official vendor documentation. Always verify current 3CX requirements and firewall ports against the official documentation for the exact product version in use.
