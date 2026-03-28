# Handoff Note

Use this as the one-message handoff for another agent owner.

```md
I packaged a portable skill for end-to-end media summary + fact-check workflows.

Files:
- `skills/mtw-portable/`
- `skills/mtw-portable.skill`

What it does:
- treats `mtw URL` as a full workflow contract
- fetches media/text source
- extracts transcript or captions
- cleans transcript text
- produces concise markdown output in this format:

**Summary**
1. ...
2. ...

**Fact-check**
1. ...
2. ...

Default behavior:
- concise output only
- full report only when explicitly requested
- saves each run locally to `mtw_runs/<human-readable-topic-slug>.md`
- supports backfilling prior runs from transcript/media artifacts

Good trigger examples:
- `mtw https://...`
- `summarize and fact-check this reel`
- `run the media transcriber workflow on this`

Install options:
1. copy the `mtw-portable/` folder into your skills directory
2. or install/import the packaged `mtw-portable.skill` file if your setup supports packaged skills
```
