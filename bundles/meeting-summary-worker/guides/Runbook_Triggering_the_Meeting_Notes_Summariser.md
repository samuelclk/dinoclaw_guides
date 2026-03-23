# Runbook: Triggering the Meeting Notes Summariser (Current Interim Workflow)

> Doc index: `workspace/guides/Meeting_Notes_Summariser_Index.md`
> GitHub: <https://github.com/stakesaurus-lido/lidoclaw/blob/main/workspace/guides/Meeting_Notes_Summariser_Index.md>

Use this runbook to trigger the Meeting Notes summarising process **right now**, before a fully persistent thread-bound specialist worker is available on every chat surface.

This is the current practical operator workflow.

---

## What this runbook is for

Use this when:
- the source meeting material has already been uploaded to the Google Drive **Source** folder
- you want Dino to run the Samuel-style meeting-summary workflow
- you want a simple, repeatable command pattern across Telegram, Discord, and other chat surfaces

---

## What Dino assumes by default

If you say:

```text
summarize <filename>
```

Dino will assume:
- the file is in the Google Drive **Source** folder
- the Samuel-style meeting-summary workflow should be used
- a matching negative example should be used if one exists
- output should be the **final summary only** unless you request more
- no target comparison should be done unless you explicitly ask for it

---

## Where files should be uploaded

## Source
Upload the input file to the Google Drive **Source** folder.

This can be:
- a raw transcript
- Gemini notes / summary
- another meeting-note input document intended for rewrite

## Negative
Upload a matching file to **Negative** when you want to show the system what bad / overlong / too-Gemini-ish output looks like.

## Target
Only upload the target later when you want:
- evaluation
- gap analysis
- scoring
- rule improvement

---

## Canonical naming pattern

Use rigid names.

### Training examples

```text
training_source_11_companyname
training_negative_example_11_companyname
training_target_11_companyname
```

### Blind tests

```text
blindtest_source_02_companyname
blindtest_negative_example_02_companyname
blindtest_target_02_companyname
```

---

# Primary operator commands

These are the standard commands to use across chat surfaces.

## 1. Summarize one file

```text
summarize <filename>
```

### Example

```text
summarize blindtest_source_01_kiln
```

Use this when you want the normal Samuel-style summary output.

---

## 2. Summarize one file and self-score it

```text
summarize <filename> and self-score it
```

### Example

```text
summarize blindtest_source_01_kiln and self-score it
```

Use this when you want:
- the final summary
- Dino’s rubric-based self-check

---

## 3. Run a blind test

```text
blind test <filename>
```

### Example

```text
blind test blindtest_source_01_kiln
```

Use this when:
- the handwritten target is not uploaded yet
- you want to test generalization first

---

## 4. Compare source against target

```text
compare <source filename> against <target filename>
```

### Example

```text
compare blindtest_source_01_kiln against blindtest_target_01_kiln
```

Use this when the handwritten target has been uploaded and you want:
- score
- gap analysis
- rule-improvement suggestions

---

## 5. Batch process multiple files

```text
summarize all new files in Source
```

or

```text
process all blind tests in Source
```

Use this when there are several new files waiting in the Source folder.

---

# Optional higher-clarity command phrasing

These longer forms are also valid and may help reduce ambiguity.

## Example forms

```text
summarize blindtest_source_01_kiln using the meeting-summary workflow
```

```text
run the Samuel-style meeting summariser on blindtest_source_01_kiln
```

```text
rewrite the Gemini notes in blindtest_source_01_kiln into my meeting-summary style
```

---

# What Dino will infer automatically

You do **not** need to tell Dino whether the source is:
- a raw transcript, or
- Gemini notes

Dino should infer this from the file contents and switch modes accordingly.

## If source is a raw transcript
Dino should:
- extract durable facts first
- then compress them into Samuel-style notes

## If source is a Gemini summary
Dino should:
- prune narrative structure
- remove Gemini-style bloat
- rewrite into Samuel-style notes

---

# Best operator habit

After uploading a file, send exactly one clean command.

## Good examples

```text
summarize blindtest_source_02_companyname
```

```text
summarize training_source_11_companyname
```

```text
compare blindtest_source_02_companyname against blindtest_target_02_companyname
```

This is the lowest-friction and least ambiguous pattern.

---

# Recommended team standard

To keep things consistent across all surfaces, standardize on this command family:

```text
summarize <file>
score <file>
compare <source> against <target>
summarize all new files in Source
```

This gives the team one repeatable language for:
- Telegram
- Discord
- any future surfaces

---

# Example workflows

## Workflow A - Normal one-off summary

1. Upload file to **Source**
2. Send:

```text
summarize training_source_11_companyname
```

3. Dino returns final summary

---

## Workflow B - Blind test

1. Upload source to **Source**
2. Upload matching negative to **Negative**
3. Do **not** upload target yet
4. Send:

```text
blind test blindtest_source_02_companyname
```

5. Review the draft
6. Later upload target and compare

---

## Workflow C - Post-hoc evaluation

1. Upload source
2. Upload target
3. Send:

```text
compare blindtest_source_02_companyname against blindtest_target_02_companyname
```

4. Dino returns:
- summary comparison
- rubric score
- gap analysis
- possible rule improvements

---

## Workflow D - Batch processing

1. Upload several files to Source
2. Send:

```text
summarize all new files in Source
```

3. Dino processes them in sequence

---

# Current limitation to understand

Right now, the specialist worker is not fully persistent on this Telegram surface because thread-bound persistent worker support is blocked by surface/platform constraints.

So the current workflow is:
- file-driven
- command-triggered
- specialist-style but not fully stateful across every surface

That means operators should be explicit and use the standard command patterns.

---

# Operational advice

## Be explicit with filenames
Avoid saying:

```text
summarize the file I just uploaded
```

unless there is only one obvious new file.

Prefer:

```text
summarize blindtest_source_01_kiln
```

## Upload targets later for blind tests
If you want a true blind evaluation, do not upload the handwritten target until after Dino has produced the first draft.

## Use matching names
Keep the same numeric prefix and slug across Source / Negative / Target files.
This makes later comparisons much easier.

---

# Minimal quickstart

If you only remember one thing, remember this:

1. Upload to **Source**
2. Send:

```text
summarize <filename>
```

That is the canonical trigger.

---

# Final summary

Until the persistent thread-bound worker path is fully live everywhere, the correct interim method is:

1. upload source material to Google Drive **Source**
2. optionally upload matching negative / target files
3. trigger Dino with a standard command like:

```text
summarize <filename>
```

or

```text
compare <source> against <target>
```

That is the current clean operator workflow for the Meeting Notes summariser.
