---
name: mtw-portable
description: Portable end-to-end media transcription, summarization, and fact-check workflow for Instagram reels, YouTube videos, TikToks, and similar media URLs. Use when a user asks for `mtw URL`, wants a media summary, transcript cleanup, concise fact-checking, caption-first extraction, or a reusable fetch→transcribe→clean→summarize→verify pipeline. Also use when the user wants a compact markdown output with `**Summary**` and `**Fact-check**` sections, local archival of results, or a portable version of the MTW workflow that another agent can install and reuse.
---

# MTW Portable

Treat `mtw URL` as a full workflow contract, not a fetch hint.

## Core contract

Execute this sequence unless the user narrows it:
1. Fetch source media or available text source.
2. Extract text from the best available source.
3. Normalize and clean the extracted text.
4. Summarize key points.
5. Fact-check key claims where possible.
6. Deliver the final package.
7. Save the final concise result locally.

Do not report success before the final deliverable exists.

## Source policy

- Try `yt-dlp` first for supported media URLs.
- Prefer captions/subtitles first when the user requested captions only, or when good captions are available and sufficient.
- If captions are unavailable or insufficient and transcription is allowed, use the host's configured transcription path.
- In this setup, prefer Deepgram before Whisper.
- Check `/home/huatyou/.openclaw/.env` for `DEEPGRAM_API_KEY` and use the Deepgram API directly when no local CLI wrapper is available.
- Do not treat a missing `deepgram` binary in `PATH` as evidence that Deepgram is unavailable.
- Do not switch to Whisper unless Stakesaurus explicitly asks for Whisper.
- Respect explicit user constraints. If the user forbids audio transcription, do not transcribe audio as fallback.

## Default output format

Unless the user explicitly asks for a full report, return only:

**Summary**
1. ...
2. ...

**Fact-check**
1. ...
2. ...

Formatting rules:
- Use markdown-compatible output.
- Use bold section titles exactly: `**Summary**` and `**Fact-check**`.
- Restart numbering from `1.` in each section.
- Use concise sentences or short phrases.
- Add line breaks for readability.

## Full report rule

Produce a larger source/transcript/artifact report only when the user explicitly asks for it.

## Progress and completion rules

If you send progress updates, label them as:
- `running`
- `blocked on X`
- `final package ready`

Do not say `done`, `complete`, or `succeeded` before the final requested deliverable exists.

## Local save rule

Also save each completed concise result locally as markdown under a human-readable topic slug:
- `mtw_runs/<human-readable-topic-slug>.md`

Prefer topic/content-based filenames over opaque video IDs unless no better name is available.

Each saved file should include:
- title when known
- source URL when known
- artifact directory when known
- `**Summary**`
- `**Fact-check**`

## Fact-check standard

Do not call something fact-checked just because it matches the transcript or caption text.
Verify key claims against external sources when possible.
If external verification is limited, say so briefly and avoid overstating certainty.

Use this fact-check sequence:
1. Identify the claims that are actually checkable.
2. Skip purely subjective opinions, personal taste, and obvious framing language unless the user asks.
3. Use web search to verify the key factual claims.
4. Compare at least one external source against the transcript/caption claim.
5. Write concise fact-check bullets that label each claim as supported, plausible but unverified, mixed, or unsupported.
6. If you could not verify a claim externally, say that briefly instead of overstating certainty.

## Backfill rule

If the user asks to backfill prior media runs, look for existing local artifacts first:
- transcript JSON
- cleaned transcript text
- metadata files
- prior downloaded media directories

Backfill concise markdown summaries conservatively when only transcript artifacts exist.

## References

Read these only if needed:
- `references/output-template.md` for the exact portable output/archive shape.
- `references/backfill-playbook.md` for how to recover prior runs from local artifacts.
