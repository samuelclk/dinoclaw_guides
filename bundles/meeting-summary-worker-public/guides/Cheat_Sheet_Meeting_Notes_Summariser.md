# Cheat Sheet: Meeting Notes Summariser

> Doc index: `bundles/meeting-summary-worker-public/guides/Meeting_Notes_Summariser_Index.md`

Use this on Slack, chat surface, or Discord.

## Upload here
- **Source** = transcript or Gemini notes
- **Negative** = bad / too-Gemini-ish version *(optional)*
- **Target** = your handwritten version *(only if you want scoring/comparison later)*

## Name files like this
- `training_source_11_companyname`
- `training_target_11_companyname`
- `training_negative_example_11_companyname`
- `blindtest_source_01_companyname`
- `blindtest_negative_example_01_companyname`
- `blindtest_target_01_companyname`

## Say this
### Normal summary
```text
summarize <filename>
```
Example:
```text
summarize blindtest_source_01_kiln
```

### Summary + self-score
```text
summarize <filename> and self-score it
```

### Compare against target
```text
compare <source> against <target>
```
Example:
```text
compare blindtest_source_01_kiln against blindtest_target_01_kiln
```

### Batch mode
```text
summarize all new files in Source
```

## Expect this
### If you say:
```text
summarize <filename>
```
the assistant will assume:
- file is in **Source**
- use the meeting-summary workflow
- use matching negative example if available
- return the **final summary only**

### If source is a raw transcript
the assistant will:
- extract the durable takeaways
- rewrite them into high-signal meeting notes

### If source is Gemini notes
the assistant will:
- prune the bloat
- rewrite into your meeting-summary style

## Blind test flow
1. Upload source to **Source**
2. Upload matching negative to **Negative** *(optional but useful)*
3. Do **not** upload target yet
4. Say:
```text
blind test <filename>
```
5. Review draft
6. Upload target later if you want comparison/scoring

## Keep in mind
- **chat surface**: good for manual triggering
- **Discord**: better later for persistent specialist lanes
- Best default command is still:
```text
summarize <filename>
```