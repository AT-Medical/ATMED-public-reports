# Notification Requirements

## Purpose

Every new or updated public report or Reddit draft should trigger an NTFY notification with a direct GitHub file link.

## Required messages

### Public report

```text
📋 Neuer öffentlicher Report:
<TITEL>

GitHub-Link: <DIREKTER_LINK_ZUR_DATEI>
```

### Reddit draft/report

```text
🌐 Neuer Reddit-Bericht:
<TITEL>

GitHub-Link: <DIREKTER_LINK_ZUR_DATEI>
```

## Rule

- Use the YAML frontmatter title where possible.
- If no title exists, use the filename.
- Always include the direct GitHub file link.
- Public report and Reddit draft notifications are separate events.

## Target

The notification target is the existing ATMED GitHub NTFY topic:

```text
ntfy.at-medical.de/atmed-github
```
