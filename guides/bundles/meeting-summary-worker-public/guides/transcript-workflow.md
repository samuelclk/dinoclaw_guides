# Transcript Summary Workflow

## Purpose
Process shared Google Docs that represent meeting transcripts, generate a concise summary, save it locally for GitHub backup, upload it to shared storage, and notify the operator in chat surface.

## Current Trigger Rule
Only treat a shared Google Doc as a meeting transcript candidate if:
- it is accessible to `<shared-service-account@example.com>`
- its title includes `AI-generated meeting notes`

## Source Selection Rule
When a matching Google Doc has multiple tabs, use:
- `Transcript` tab as the source for summarization

Do not summarize from the `Notes` tab.

## Storage locations
Root storage folder:
- `<org>`

Managed transcript copies:
- `<org>/Meeting Transcripts`

Final summaries:
- `<org>/Meeting Notes`

## Trigger Mode
Manual first.

Expected user command pattern:
- `process latest meeting notes`
- or similar explicit instruction from the operator

## Candidate Selection Logic
On manual trigger:
1. Search accessible Google Docs for titles containing `AI-generated meeting notes`
2. Filter to Google Docs only
3. Sort by recency
4. Exclude any source doc IDs already present in the dedupe log
5. Select the newest remaining candidate

## Dedupe Rule
Track processed source document IDs in:
- `workspace/outputs/meeting-summary-log.json`

Each log entry should include:
- `sourceDocId`
- `sourceTitle`
- `copiedSourceDocId`
- `copiedSourceTitle`
- `summaryFile`
- `summaryDriveFileId`
- `processedAt`
- `sourceTabUsed` (expected: `Transcript`)
- `gitCommit`

If a source document ID is already present in the log, skip it unless the operator explicitly asks for a rerun.

## Copy-First Rule
Do not summarize directly from the original shared doc.

Instead:
1. Copy the matching source doc into `<org>/Meeting Transcripts`
2. Use the copied doc’s `Transcript` tab as the summarization source

This keeps the assistant’s workflow inside the approved Drive boundary.

## Summary Output Format
For now, the summary should include only:
- title
- participants
- key discussion points
- next steps

Recommended markdown structure:

```md
# <title>

## Participants
- ...

## Key Discussion Points
- ...
- ...

## Next Steps
- ...
- ...
```

## Output Naming Convention
Use:
- `<original title> - Summary.md`

The original title is expected to already contain the useful timestamp.

## Local Output Path
Save summaries to:
- `workspace/outputs/`

## GitHub Backup Rule
After writing the summary file:
1. commit the new/updated output file
2. push to `origin/main`

## shared storage Upload Rule
After local save + git push:
1. upload the summary file to `<org>/Meeting Notes`
2. capture the resulting storage file URL

## chat surface Notification Rule
Notify the operator in the chat surface group under the `Meeting Notes` topic.

Recommended short format:
- `Meeting summary ready: <title>`
- `Saved locally and pushed to GitHub`
- `Uploaded to <org>/Meeting Notes`
- `<storage URL>`

## Known Missing Piece
The chat surface topic ID for `Meeting Notes` still needs to be recorded if not discoverable automatically from current tooling.

## End-to-End Procedure
1. the operator sends manual trigger in chat
2. the assistant finds newest unprocessed Google Doc whose title contains `AI-generated meeting notes`
3. the assistant copies it into `<org>/Meeting Transcripts`
4. the assistant reads the copied doc’s `Transcript` tab
5. the assistant generates a summary with:
   - title
   - participants
   - key discussion points
   - next steps
6. the assistant saves the markdown file under `workspace/outputs/`
7. the assistant commits and pushes to GitHub
8. the assistant uploads the summary file to `<org>/Meeting Notes`
9. the assistant sends chat surface notification in `Meeting Notes` topic
10. the assistant appends an entry to `workspace/outputs/meeting-summary-log.json`