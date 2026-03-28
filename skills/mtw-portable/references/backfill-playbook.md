# Backfill Playbook

Use this when a user asks to recover or backfill MTW outputs from prior local artifacts.

## Artifact search order

Search for these first:
1. `meta.json`
2. `deepgram.json`
3. `deepgram-response.json`
4. `cleaned_transcript.txt`
5. downloaded media files

Common locations:
- `tmp/`
- `tmp_media_transcriber/`
- `instagram_transcripts/`
- `youtube_transcripts/`

## Backfill steps

1. Inventory candidate artifact directories.
2. Prefer runs with both metadata and transcript files.
3. Extract the title/source URL when available.
4. Read the transcript or cleaned transcript.
5. Produce concise `**Summary**` and `**Fact-check**` sections.
6. Save the result to `mtw_runs/<human-readable-topic-slug>.md`.

## Confidence rules

- If only transcript artifacts exist, summarize conservatively.
- If external fact-checking was not performed, say that briefly in the fact-check section.
- Do not overclaim source certainty when the original URL/title is missing.

## Batch behavior

When backfilling many runs:
- batch discovery first
- then serialize write operations
- avoid unnecessary duplicate rewrites
