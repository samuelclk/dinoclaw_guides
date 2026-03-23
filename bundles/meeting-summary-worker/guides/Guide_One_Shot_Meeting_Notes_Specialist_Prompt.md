# Guide: One-Shot Meeting Notes Specialist Prompt

Use this guide when the Meeting Notes workflow is **not automatically using the trained style files**, and you want to force a **one-shot specialist pass** from a chat surface like the Meeting Notes Telegram topic.

This is the interim manual override method.

---

## When to use this

Use this when:
- the source file is already uploaded to the Google Drive **Source** folder
- Dino is not reliably using the meeting-summary training results by default
- you want to force the summariser to use the canonical style/prompt/rules files
- you want a one-shot specialist rewrite without waiting for persistent worker routing

---

## Goal

Force Dino to:
- load the canonical meeting-summary training files
- use the matching Negative example if available
- rewrite the source into Samuel-style high-signal meeting notes
- return the final summary in the learned house style

---

## Full prompt template

Copy and paste this into the Meeting Notes topic, replacing `<FILENAME>`.

```text
Run a one-shot meeting-summary specialist pass on <FILENAME>.

Use the canonical meeting-summary training files in the workspace:
- workspace/meeting-summary-style-training/production-prompt.md
- workspace/meeting-summary-style-training/style-spec.md
- workspace/meeting-summary-style-training/rewrite-rules.md
- workspace/meeting-summary-style-training/scoring-rubric.md

Task:
1. Load the source file from the Google Drive Source workflow / local mirrored source for <FILENAME>
2. If a matching Negative example exists, use it as an anti-pattern reference
3. Rewrite the source into Samuel-style high-signal meeting notes
4. Focus on:
   - counterpart
   - real opportunity / blocker / dependency
   - key discussion points worth retaining
   - concrete next steps
5. Remove:
   - chronology
   - greetings
   - travel/social chatter
   - long product explanations
   - generic Gemini-style bloat

Output requirements:
- Return final summary only
- Use the learned Samuel-style meeting-notes format
- Prefer compact internal operator notes over meeting minutes

If the source is a raw transcript, extract then rewrite.
If the source is Gemini notes, prune then rewrite.
```

---

## Short prompt template

Use this when you want the lighter version.

```text
Run a one-shot meeting-summary specialist pass on <FILENAME> using:
- workspace/meeting-summary-style-training/production-prompt.md
- workspace/meeting-summary-style-training/style-spec.md
- workspace/meeting-summary-style-training/rewrite-rules.md
- workspace/meeting-summary-style-training/scoring-rubric.md

Use matching Negative if available. Return final Samuel-style summary only.
```

---

## Prompt template with self-score

Use this if you want the summary plus a rubric-based self-check.

```text
Run a one-shot meeting-summary specialist pass on <FILENAME> using the canonical meeting-summary training files:
- workspace/meeting-summary-style-training/production-prompt.md
- workspace/meeting-summary-style-training/style-spec.md
- workspace/meeting-summary-style-training/rewrite-rules.md
- workspace/meeting-summary-style-training/scoring-rubric.md

Use matching Negative if available.
Return:
1. final Samuel-style summary
2. brief rubric self-score
```

---

## Concrete example

For the Kiln blind test, use:

```text
Run a one-shot meeting-summary specialist pass on blindtest_source_01_kiln.

Use the canonical meeting-summary training files in the workspace:
- workspace/meeting-summary-style-training/production-prompt.md
- workspace/meeting-summary-style-training/style-spec.md
- workspace/meeting-summary-style-training/rewrite-rules.md
- workspace/meeting-summary-style-training/scoring-rubric.md

Use the matching Negative example if available.
Return final Samuel-style summary only.
```

---

## Why this works better than a vague prompt

Right now, the workflow may not auto-route into the trained meeting-summary system consistently.

This prompt fixes that by explicitly forcing:
- the correct style files
- the correct rewrite rules
- the right anti-pattern handling
- the right output format

It reduces the chance of getting generic AI meeting minutes instead of the trained summary style.

---

## Best practice

If the normal trigger:

```text
summarize <filename>
```

is producing generic output, switch immediately to this one-shot specialist prompt.

That is the current clean manual override.

---

## Recommended operator pattern

### Normal case

```text
summarize <filename>
```

### If the output ignores the trained style

Use the one-shot specialist prompt from this guide.

### If you want scoring

Use the self-score version.

---

## Minimal summary

If you only remember one thing, remember this:

When Dino is not using the learned meeting-summary style correctly, paste the explicit one-shot specialist prompt with:
- `production-prompt.md`
- `style-spec.md`
- `rewrite-rules.md`
- `scoring-rubric.md`

That forces the trained workflow back into play.
