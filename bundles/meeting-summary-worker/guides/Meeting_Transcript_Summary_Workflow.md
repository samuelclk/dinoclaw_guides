# Meeting Transcript Summary Workflow

## Purpose
Process shared Google Docs that represent meeting transcripts, generate a concise summary, save it locally for GitHub backup, upload it to Google Drive, and notify Stakesaurus in Telegram.

## Current Trigger Rule
Only treat a shared Google Doc as a meeting transcript candidate if:
- it is accessible to `dino@stakesaurus.com`
- its title includes `Notes by Gemini`

## Source Selection Rule
When a matching Google Doc has multiple tabs, use:
- `Transcript` tab as the source for summarization

Do not summarize from the `Notes` tab.

## Drive Locations
Root workspace folder:
- `Lido` — `15BjyVQE-6DsrBL3nz-tE2lzNfvaFmRVj`

Managed transcript copies:
- `Lido/Meeting Transcripts` — `1ihgS4J7Z155aFz12OAmOLNkC8Ke93ZUT`

Final summaries:
- `Lido/Meeting Notes` — `1rlBc-SFcR8nsNqtpZXF6EwEwMgWbG3Bh`

## Trigger Mode
Manual first.

Expected user command pattern:
- `process latest meeting notes`
- or similar explicit instruction from Stakesaurus

## Candidate Selection Logic
On manual trigger:
1. Search accessible Google Docs for titles containing `Notes by Gemini`
2. Filter to Google Docs only
3. Sort by recency
4. Exclude any source doc IDs already present in the dedupe log
5. Select the newest remaining candidate

## Dedupe Rule
Track processed source document IDs in:
- `workspace/outputs/meeting-transcript-log.json`

Each log entry should include:
- `sourceDocId`
- `sourceTitle`
- `copiedTranscriptDocId`
- `copiedTranscriptTitle`
- `summaryFile`
- `summaryDriveFileId`
- `processedAt`
- `sourceTabUsed` (expected: `Transcript`)
- `gitCommit`

If a source document ID is already present in the log, skip it unless Stakesaurus explicitly asks for a rerun.

## Copy-First Rule
Do not summarize directly from the original shared doc.

Instead:
1. Copy the matching source doc into `Lido/Meeting Transcripts`
2. Use the copied doc’s `Transcript` tab as the summarization source

This keeps Dino’s workflow inside the approved Drive boundary.

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

## Google Drive Upload Rule
After local save + git push:
1. upload the summary file to `Lido/Meeting Notes`
2. capture the resulting Drive file URL

## Telegram Notification Rule
Notify Stakesaurus in the Telegram group under the `Meeting Notes` topic.

Recommended short format:
- `Meeting summary ready: <title>`
- `Saved locally and pushed to GitHub`
- `Uploaded to Lido/Meeting Notes`
- `<Drive URL>`

## Known Missing Piece
The Telegram topic ID for `Meeting Notes` still needs to be recorded if not discoverable automatically from current tooling.

## End-to-End Procedure
1. Stakesaurus sends manual trigger in chat
2. Dino finds newest unprocessed Google Doc whose title contains `Notes by Gemini`
3. Dino copies it into `Lido/Meeting Transcripts`
4. Dino reads the copied doc’s `Transcript` tab
5. Dino generates a summary with:
   - title
   - participants
   - key discussion points
   - next steps
6. Dino saves the markdown file under `workspace/outputs/`
7. Dino commits and pushes to GitHub
8. Dino uploads the summary file to `Lido/Meeting Notes`
9. Dino sends Telegram notification in `Meeting Notes` topic
10. Dino appends an entry to `workspace/outputs/meeting-transcript-log.json`
