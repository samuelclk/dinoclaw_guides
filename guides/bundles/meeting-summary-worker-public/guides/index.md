# Meeting Summary Workflow - Index

This is the index page for the Meeting Summary Workflow documentation set.

Use it as the entry point for operators, teammates, and future setup work.

---

## Recommended reading order

### 1. Cheat Sheet
Fastest possible version.
Use this if you just want:
- where to upload files
- what command to send
- what output to expect

- Local: `bundles/meeting-summary-worker-public/guides/cheat-sheet.md`

### 2. Quickstart
Shortest path to replicate the setup.
Use this if you want the minimum viable system.

- Local: `bundles/meeting-summary-worker-public/guides/quickstart.md`

### 3. Operator Guide
Full build methodology and replication logic.
Use this if you want the whole system, not just the minimum steps.

- Local: `bundles/meeting-summary-worker-public/guides/operator-guide.md`

### 4. Trigger Runbook
How to actually trigger the summarizer in the current interim workflow.
Use this for day-to-day operation.

- Local: `bundles/meeting-summary-worker-public/guides/runbook.md`

### 5. Discord Setup Guide
How to configure Discord for persistent thread-bound specialist workers.
Use this if you want the best long-term chat surface for this workflow.

- Local: `bundles/meeting-summary-worker-public/guides/discord-setup.md`

---

## What each document is for

### Cheat Sheet
Audience:
- anyone who just needs to use the system

Contains:
- upload here
- say this
- expect this

### Quickstart
Audience:
- teammate setting up the minimum workflow

Contains:
- folder structure
- dataset pattern
- standard commands
- blind test flow

### Operator Guide
Audience:
- system owner / workflow designer

Contains:
- architecture
- design rationale
- dataset structure
- style distillation method
- blind testing method
- cross-surface lessons
- replication checklist

### Trigger Runbook
Audience:
- day-to-day operator

Contains:
- exact command patterns
- how to trigger on current surfaces
- blind test flow
- compare flow
- batch flow

### Discord Setup Guide
Audience:
- operator enabling persistent specialist workflows

Contains:
- bot setup
- `.env` token handling
- allowlists
- thread bindings
- ACP / subagent thread-bound worker setup
- troubleshooting

---

## Minimal recommended path for new teammates

If someone is brand new, tell them to read in this order:

1. **Cheat Sheet**
2. **Quickstart**
3. **Trigger Runbook**

Only read the full **Operator Guide** if they need to understand or modify the system.
Only read the **Discord Setup Guide** if they are setting up the long-term specialist-worker surface.

---

## Core supporting files

These are the durable style/system files behind the workflow:

- `bundles/meeting-summary-worker-public/templates/style-guide.md`
- `bundles/meeting-summary-worker-public/templates/rewrite-rules.md`
- `bundles/meeting-summary-worker-public/templates/rubric.md`
- `bundles/meeting-summary-worker-public/templates/prompt.md`
- `bundles/meeting-summary-worker-public/templates/worker-spec.md`
- `bundles/meeting-summary-worker-public/templates/inputs-outputs.md`

---

## Current operational note

At the moment:
- chat surface works well for manual triggering
- Discord is the better long-term home for persistent thread-bound specialist workflows
- the style expertise is intentionally stored in files so the workflow survives resets, model changes, and surface changes

---

## Summary

If you only want one landing page, this is it.

Start with:
- **Cheat Sheet** for usage
- **Quickstart** for setup
- **Operator Guide** for full replication
- **Trigger Runbook** for daily operation
- **Discord Setup Guide** for long-term specialist-worker deployment