---
title: "3CX manual VPS install: temporary port 5015 and configuration file workflow gotchas"
source_public_report: "incident-reports/communication/2026-05-07_3cx-hetzner-installation-troubleshooting.md"
category: "communication"
tags:
  - "3cx"
  - "vps"
  - "pbx"
  - "voip"
  - "debian"
  - "selfhosted"
hashtags:
  - "#3CX"
  - "#VoIP"
  - "#SelfHosted"
  - "#VPS"
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
    - "r/3CX"
    - "r/selfhosted"
    - "r/sysadmin"
    - "r/VOIP"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Draft Reddit Post

## Suggested title

3CX manual VPS install: temporary port 5015 and configuration file workflow gotchas

## Suggested subreddits and rationale

- `r/3CX`: most directly relevant to 3CX PBX setup behavior.
- `r/selfhosted`: useful for admins deploying PBX software on their own VPS.
- `r/sysadmin`: relevant due to firewall, deployment workflow and operations implications.
- `r/VOIP`: relevant for PBX/VoIP administrators and SIP infrastructure discussions.

## Post body

I ran into a couple of practical gotchas while setting up a 3CX PBX manually on a Debian-based VPS, so I’m sharing the sanitized version in case it helps someone else.

The main issue was that the local 3CX web configuration tool starts on TCP port 5015. If that port is blocked by the provider firewall, the setup URL simply times out in the browser, even though the installer itself has completed correctly. Temporarily allowing TCP 5015 during setup fixed the first blocker.

The second issue was workflow-related: the hosted/guided cloud deployment path in the 3CX portal is not the same as a generic manual VPS install. For a manual Linux VPS installation, the useful path is the one that generates a configuration setup file, which is then uploaded to the local 3CX configuration page on the VPS.

The rough working sequence was:

1. Install 3CX on the Debian-based VPS.
2. Start the browser-based 3CX configuration tool.
3. Temporarily allow TCP 5015 in the provider firewall.
4. Open the local setup URL via HTTP on port 5015.
5. Generate/download the correct manual Linux/on-premise configuration file from the 3CX portal.
6. Upload that file to the local setup tool.
7. Complete setup and verify the Admin Console.
8. Close TCP 5015 again after setup.
9. Run the 3CX firewall checker and configure final PBX ports.
10. Configure remote backups before using it seriously.

The most important lesson for me: don’t leave port 5015 open after setup, and don’t confuse hosted PBX provisioning with a manual VPS/self-managed installation.

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de
