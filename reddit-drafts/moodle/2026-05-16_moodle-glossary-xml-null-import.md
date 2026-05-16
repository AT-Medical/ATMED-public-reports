---
title: "Moodle glossary XML import accepted the file but imported 0 entries — fixed by matching the instance export schema"
source_public_report: "incident-reports/moodle/2026-05-16_moodle-glossary-xml-null-import.md"
category: "moodle"
tags:
  - "moodle"
  - "xml"
  - "glossary"
  - "selfhosted"
hashtags:
  - "#Moodle"
  - "#XML"
  - "#SelfHosted"
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
    - "r/moodle"
    - "r/selfhosted"
    - "r/edtech"
    - "r/sysadmin"
  selected_subreddit: null
  flair: null
  posted_url: null
  posted_at: null
---

# Draft Reddit Post

## Suggested subreddits and rationale

- **r/moodle**: Most specific audience for Moodle import/export behavior.
- **r/selfhosted**: Useful for administrators running their own Moodle instances.
- **r/edtech**: Relevant for course-content and LMS workflows.
- **r/sysadmin**: Potentially useful where admins maintain Moodle and troubleshoot silent import failures.

## Title

Moodle glossary XML import accepted the file but imported 0 entries — fixed by matching the instance export schema

## Body

I ran into a small but annoying Moodle glossary import issue that may help someone else.

I generated a Moodle glossary XML file programmatically. Moodle accepted the upload in the glossary import screen, but the import result showed no usable entries. There was no classic XML parse error, so at first glance the file looked valid.

The fix was not just “make the XML well-formed.” The important part was matching the exact glossary export structure produced by the target Moodle instance.

The process that solved it:

1. Create two simple glossary entries manually in Moodle.
2. Export that glossary from Moodle.
3. Use that export as the canonical schema reference.
4. Regenerate the full XML import using the same structure.
5. Test with a tiny import file first.
6. Then import the full glossary.

The key lesson: Moodle glossary XML can be structurally sensitive. A file may be valid XML and still import as zero entries if it does not match the expected glossary import/export structure closely enough.

In the working case, the entries were nested in the info block, and definition content matched Moodle's own exported representation as escaped HTML inside the definition element. After mirroring the Moodle-generated export, the import worked.

Transparency note: This is an automatically generated incident report based on a real troubleshooting and remediation process performed with the assistance of ChatGPT. The summary was generated and prepared through GitHub by an automated worker and was published only after approval by the IT and Security department of AT Medical GmbH, Heidelberg.

Organization: AT Medical GmbH, Heidelberg  
Website: https://www.at-medical.de  
Contact: Support@at-medical.de
