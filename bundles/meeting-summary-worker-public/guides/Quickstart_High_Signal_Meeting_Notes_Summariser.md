# Quickstart: High-Signal Meeting Notes Summariser

> Doc index: `bundles/meeting-summary-worker-public/guides/Meeting_Notes_Summariser_Index.md`

This is the shortest path to replicate the meeting-summary workflow.

Use this if you only want the minimum setup steps.

---

## Goal

Turn:
- raw meeting transcripts, or
- Gemini meeting notes

into:
- high-signal internal notes
- key discussion points
- concrete next steps
- a consistent human-style summary format

---

## Step 1 - Create 3 folders

Create these folders locally and in shared storage:

```text
Source
Target
Negative
```

### Purpose
- **Source** = transcript or Gemini notes
- **Target** = handwritten ideal summary
- **Negative** = bad / verbose / too-Gemini-ish example

---

## Step 2 - Collect 5-15 examples

Best pattern:
- source = raw transcript
- target = your handwritten final summary
- negative = Gemini summary

Fallback pattern:
- source = Gemini summary
- target = your handwritten final summary
- negative = same Gemini summary or another bad example

---

## Step 3 - Use rigid names

Examples:

```text
training_source_1_companyname
training_target_1_companyname
training_negative_example_1_companyname
```

Blind tests:

```text
blindtest_source_01_companyname
blindtest_negative_example_01_companyname
blindtest_target_01_companyname
```

---

## Step 4 - Create the core files

Minimum files to maintain:

- `style-spec.md`
- `rewrite-rules.md`
- `scoring-rubric.md`
- `production-prompt.md`

These files are the durable brain of the system.

---

## Step 5 - Teach the right style

The target style should be:
- compressed
- operator-facing
- high-signal
- non-chronological
- focused on what matters next

Drop:
- greetings
- audio glitches
- travel chatter
- long product explanations
- generic AI fluff

Keep:
- counterpart
- real opportunity
- blocker / dependency
- useful numbers / timing
- concrete next steps

---

## Step 6 - Use the standard commands

After uploading a file to **Source**, trigger with:

```text
summarize <filename>
```

Examples:

```text
summarize blindtest_source_01_kiln
summarize training_source_8_hashkey
```

Other useful commands:

```text
summarize <filename> and self-score it
compare <source> against <target>
summarize all new files in Source
```

---

## Step 7 - Run blind tests

Blind test flow:
1. upload source to **Source**
2. upload matching negative to **Negative**
3. do **not** upload target yet
4. run:

```text
blind test blindtest_source_01_kiln
```

5. review the output
6. upload target later
7. compare against target and tighten the rules

---

## Step 8 - Keep the expertise in files, not memory

Do not rely on chat history or model memory.
Keep the workflow in version-controlled files.

Commit these:
- prompt
- style spec
- rewrite rules
- rubric
- operator guides

Do **not** commit:
- `.env`
- tokens
- secrets

---

## Step 9 - Pick the right chat surface

- **chat surface** = fine for manual triggering
- **Discord** = better if you want persistent thread-bound specialist workers later

If your team wants long-lived specialist lanes, use Discord.

---

## Minimal team workflow

1. Upload transcript/notes to **Source**
2. Say:

```text
summarize <filename>
```

3. Review output
4. Add target later if you want evaluation
5. Improve the style files when failures repeat

---

## Final takeaway

The winning pattern is:

1. collect paired examples
2. distill the style into files
3. use Source / Target / Negative
4. validate with blind tests
5. keep improving the rules

That is the fastest and cleanest way to build a high-signal meeting notes summariser in OpenClaw.