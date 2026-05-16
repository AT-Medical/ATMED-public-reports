---
title: "Moodle Glossary XML Import Shows Null Entries"
date: "2026-05-16"
category: "moodle"
classification: "public"
sensitivity: "low"
status: "draft"
tags:
  - "moodle"
  - "glossary"
  - "xml"
  - "troubleshooting"
---

# Moodle Glossary XML Import Shows Null Entries

## Summary

A generated Moodle glossary XML file was accepted by Moodle's glossary import form, but Moodle reported zero/null detected entries and imported nothing. The issue was resolved by comparing the generated file against a glossary XML export created by the same Moodle instance and regenerating the import file with the same structure.

## Environment

- Platform: Moodle
- Feature: Glossary activity
- Function: Import entries from XML
- Content type: Glossary entries and abbreviation-list entries

## Problem

The import UI accepted the XML upload, but the result showed null or zero detected/imported entries. This can happen when the XML is well-formed but does not match the structure Moodle expects for glossary import.

## Observed Behavior

- XML upload did not produce a classic parse error.
- Moodle did not detect entries.
- The import summary showed no usable entries.

## Root Cause

The generated XML structure differed from the actual glossary XML export structure used by the target Moodle instance. The most important compatibility points were:

- The entries block must be nested in the info block.
- The definition field should match Moodle's exported representation.
- In the verified working case, definition content was stored as escaped HTML directly inside the definition element.
- Additional structures such as nested text elements, CDATA wrappers, categories, or aliases may not be accepted unless they match the export format of the same Moodle instance.

## Fix / Resolution

The fix was to:

1. Create one or two glossary entries manually in Moodle.
2. Export that glossary from Moodle.
3. Use the exported file as the canonical schema reference.
4. Regenerate the full import file to match that structure exactly.
5. First test with a minimal file containing only a few entries.
6. Import the full glossary only after the test file succeeds.

## Verification

The corrected file was imported successfully after it was rebuilt to match the Moodle export schema.

## Prevention / Hardening

- Always generate Moodle glossary imports from an export-derived template.
- Keep a minimal test XML file with two or three entries.
- Avoid adding optional structures until they have been tested with an export from the same Moodle instance.
- Treat Moodle glossary XML as instance-sensitive rather than assuming any generic XML shape will work.

## Lessons Learned

For Moodle glossary imports, well-formed XML is not sufficient. The XML must match the exact structure expected by the Moodle glossary import parser. The safest reference is a real export from the target Moodle instance.

## Disclaimer

This public report is generalized and sanitized. It intentionally omits organization-specific infrastructure details and private file names.
