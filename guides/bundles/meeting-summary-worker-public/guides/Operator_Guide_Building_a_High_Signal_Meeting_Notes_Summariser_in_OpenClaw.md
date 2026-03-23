# Operator Guide: Building a High-Signal Meeting Notes Summariser in OpenClaw

> Doc index: `bundles/meeting-summary-worker-public/guides/Meeting_Notes_Summariser_Index.md`

This guide distills the full setup process for building a **high-signal meeting notes summariser** that rewrites raw transcripts or AI-generated notes into a compact internal style.

It is written so a team can replicate the workflow without re-discovering all the decisions from scratch.

---

## What this system is for

Use this setup when you want OpenClaw to turn:
- raw meeting transcripts, or
- AI-generated meeting notes (for example AI summaries)

into:
- **compressed internal operator notes**
- **high-signal discussion points**
- **concrete next steps**
- a style that matches a specific human operator rather than generic AI meeting minutes

This system is especially useful when the target style values:
- retrieval over publication
- actionability over completeness
- business framing over chronology
- signal over transcript fidelity

---

# Part 1 - Core design principle

## The most important lesson

Do **not** start by “training the model.”

Start by building a **rewrite system**.

That means:
1. collect paired examples
2. extract style rules
3. write a durable prompt/spec
4. validate on blind tests
5. improve the rules over time

This approach is cheaper, faster, more controllable, and easier to maintain than jumping straight to fine-tuning.

---

# Part 2 - The architecture

The system has 4 layers.

## Layer 1 - Dataset
Three folders:
- **Source** = raw transcript or AI-generated notes
- **Target** = handwritten target summary in the desired style
- **Negative** = bad outputs / anti-pattern examples / overly verbose AI notes

## Layer 2 - Style distillation
Durable files that capture the style:
- style spec
- rewrite rules
- scoring rubric
- production prompt

## Layer 3 - Evaluation workflow
Two validation modes:
- blind test without target
- compare-against-target after target is added

## Layer 4 - Operational surface
The system can be used from:
- chat surface
- Discord
- other chat surfaces
- manual Drive-drop workflows

The style expertise should live in **files**, not only in model memory.

---

# Part 3 - Folder structure

## Recommended local structure

```text
bundles/meeting-summary-worker-public/templates/
  README.md
  source/
  target/
  negative/
  downloaded/
  blindtests/
  production-prompt.md
  style-spec.md
  rewrite-rules.md
  scoring-rubric.md
  patterns-detected.md
  subagent-spec.md
  io-contract.md
```

## Recommended shared storage structure

Inside your allowed Drive workspace:

```text
Meeting Summary Style Training/
  Source/
  Target/
  Negative/
```

## Folder purpose

### Source
Contains:
- raw transcript docs, or
- AI summaries used as rewrite input

### Target
Contains:
- handwritten final summaries in the desired house style

### Negative
Contains:
- bad AI outputs
- verbose AI summaries
- anti-pattern examples the system should avoid imitating

---

# Part 4 - Naming conventions

Use rigid names so pairing is easy.

## Training examples

```text
training_source_1_example-co
training_target_1_example-co
training_negative_example_1_example-co
```

## Blind tests

```text
blindtest_source_01_example-co
blindtest_negative_example_01_example-co
blindtest_target_01_example-co
```

## Why this matters
This makes it easy to:
- pair source/target/negative files
- automate exports and comparisons
- run blind tests without confusion

---

# Part 5 - What examples to collect

## Best starting dataset

Start with **5-15 paired examples**.

For each example, ideally collect:
- source input
- handwritten target
- negative example

## Strongest supervision pattern

Best case:
- **raw transcript** -> **handwritten target**
- **AI summary** as negative example

This teaches both:
- extraction from messy source
- stylistic rewriting away from generic AI output

## Useful fallback pattern

If raw transcript is not available:
- **AI summary** -> **handwritten target**
- AI summary can also serve as the negative baseline

This mainly teaches rewrite and pruning.

## Key insight
A mixed dataset is still useful.
You can learn:
- transcript extraction from the raw examples
- AI-to-human rewrite style from all examples

---

# Part 6 - How to extract the style

## What to infer from the examples

The summariser should learn:
- output structure
- tone and compression level
- what counts as a key discussion point
- what counts as a next step
- what gets dropped as noise
- how to phrase blockers, dependencies, and opportunities

## The target style in this build

The learned style here was:
- direct
- compressed
- operator-facing
- not chronological
- not beginner-friendly
- focused on the few durable takeaways

High-signal summaries preserved:
- partner identity / role
- opportunity angle
- blocker / dependency
- relevant sizing or timing
- concrete next steps

Low-value details were dropped:
- greetings
- audio glitches
- travel chat
- transcript chronology
- long product explanations
- descriptive company background that did not change action

---

# Part 7 - Canonical style files to create

These are the minimum durable files.

## 1. `style-spec.md`
Purpose:
- define the style at a high level
- capture tone, format, inclusion/exclusion rules

## 2. `rewrite-rules.md`
Purpose:
- turn the examples into actionable transformation rules
- raw transcript -> target
- AI summary -> target
- negative example -> target

## 3. `scoring-rubric.md`
Purpose:
- define how outputs are judged
- make review faster and more objective

## 4. `production-prompt.md`
Purpose:
- the actual reusable prompt used to generate summaries
- should only contain the minimal instructions needed for production use

## 5. `patterns-detected.md`
Purpose:
- summarize the main patterns the dataset is teaching
- useful for operator review and future improvements

---

# Part 8 - Key style lessons learned in this build

## 1. Rewrite for retrieval, not publication
The summaries should help the operator remember:
- what the meeting was about
- what mattered
- what to do next

## 2. Compression is severe, not mild
Targets were often ~700 to ~1,700 characters.
Negative examples were often ~6,000 to ~9,000 characters.
That is not a small style difference. It is a different objective.

## 3. Preserve pivots
When the real value of the meeting is a redirect from one angle to a better angle, preserve that pivot explicitly.

## 4. Prefer blocker diagnosis over vague demand language
Instead of “low demand,” rewrite toward the actual reason adoption is weak if the source supports it.

## 5. Prefer dependency logic over logistics
Preserve the condition that determines whether collaboration is worth doing.

## 6. Prune housekeeping asks
AI often promotes one-pagers, extra intros, and procedural follow-ups into major action items. Keep them only if they materially change progress.

---

# Part 9 - Two valid output formats

The learned system supported two main formats.

## Format A - Compact call summary
Use when the meeting is a straightforward 1:1 or lighter BD call.

Pattern:

```text
Call with <name> (<role>):
1. ...
2. ...
3. ...
Next steps:
1. ...
2. ...
```

## Format B - Structured meeting note
Use when the meeting is more formal, multi-party, or worth preserving in a more archival structure.

Pattern:

```text
Meeting with <org> & <org>
Participants:
* ...
* ...
________________

Notes
* ...
* ...
________________

Next Steps
* ...
* ...
```

---

# Part 10 - Production workflow

## Recommended generation flow

### For raw transcripts
Use a two-pass mental workflow:
1. extract durable facts
2. compress them into house style

### For AI summaries
Use a rewrite-and-prune workflow:
1. strip AI narrative structure
2. compress into house style

## Operator commands
Adopt a simple command family across chat surfaces:

```text
summarize <file>
score <file>
compare <source> against <target>
summarize all new files in Source
```

Examples:

```text
summarize blindtest_source_01_example-co
summarize blindtest_source_01_example-co and self-score it
compare blindtest_source_01_example-co against blindtest_target_01_example-co
```

---

# Part 11 - Blind testing

## Why blind tests matter
A system that reproduces training examples is not yet proven.
Blind tests show whether the summariser generalizes.

## Blind test procedure

1. upload a new source file to `Source`
2. optionally upload a matching negative example to `Negative`
3. do **not** upload the handwritten target yet
4. run:

```text
summarize blindtest_source_01_example-co
```

5. review the draft
6. later upload the handwritten target to `Target`
7. compare draft vs target
8. update rules if failure modes repeat

## What a blind test should measure
- signal selection
- compression quality
- business framing
- next-step quality
- tone match

---

# Part 12 - Scoring rubric design

Use a 1-5 scale on each category.

Recommended categories:
- counterparty clarity
- signal selection
- business framing
- compression quality
- next-step quality
- tone match

Interpretation band example:
- **27-30** = ready to use
- **23-26** = good, minor edits only
- **18-22** = usable but needs tightening
- **12-17** = weak
- **<12** = rewrite recommended

---

# Part 13 - Specialist worker design

## Best design principle
Keep the expertise in **files**, not inside a fragile one-off session.

That means the worker should rely on:
- `production-prompt.md`
- `style-spec.md`
- `rewrite-rules.md`
- `scoring-rubric.md`
- `subagent-spec.md`
- `io-contract.md`

## Why this matters
Then the workflow survives:
- resets
- model changes
- session changes
- different chat surfaces
- future operators

## Worker responsibilities
A specialist meeting-summary worker should:
- read source files
- produce house-style summaries
- optionally self-score
- optionally compare against target
- suggest rule updates when patterns repeat

---

# Part 14 - Cross-surface operational lesson

## Critical lesson
Do not rely on a persistent worker being available on every surface.

Persistent thread-bound specialist workers depend on the messaging surface.

### What happened in this build
- chat surface topic here did **not** support the thread-binding behavior needed for persistent sub-agent / ACP specialist sessions
- Discord docs explicitly support thread bindings and persistent ACP channel bindings

## Operational conclusion
- **chat surface** can still drive the workflow manually
- **Discord** is the better long-term home for persistent thread-bound specialist workers

---

# Part 15 - Discord as the preferred specialist surface

If the team wants:
- persistent specialist channels
- thread-bound workers
- persistent ACP worker sessions
- topic-specific lanes

then Discord is the recommended surface.

## Recommended Discord setup
- private Discord server
- one dedicated channel or forum lane per specialist workflow
- allowlisted guild
- allowlisted channel
- allowlisted user(s)
- env-backed `DISCORD_BOT_TOKEN`
- thread bindings enabled
- optional persistent ACP channel binding

Relevant companion guide:
- `bundles/meeting-summary-worker-public/guides/Discord_Setup_for_Persistent_Thread_Bound_Subagents_on_OpenClaw.md`

---

# Part 16 - Security and access control

## Secure baseline
Use:
- env-backed secrets
- allowlisted server/channel/user IDs
- `groupPolicy: "allowlist"`
- no public posting of tokens
- least-privilege bot permissions

## Team setup note
If multiple operators need access, add them deliberately by user ID.
Do not leave the bot open just because it sits in a private server.

---

# Part 17 - Practical rollout plan for a team

## Phase 1 - Build the style system
1. create Source / Target / Negative folders
2. collect 5-15 examples
3. write style spec, rewrite rules, rubric, production prompt
4. commit and push

## Phase 2 - Validate
1. run blind tests
2. compare against targets
3. document repeated failure modes
4. tighten the rules

## Phase 3 - Operationalize
1. create specialist worker spec
2. define inputs/outputs contract
3. decide the primary chat surface
4. wire the workflow to that surface

## Phase 4 - Scale to team use
1. publish the operator guides
2. standardize command phrases
3. set access control
4. keep adding good examples and negative examples over time

---

# Part 18 - Recommended file outputs to keep in version control

These are worth committing:
- `bundles/meeting-summary-worker-public/templates/README.md`
- `bundles/meeting-summary-worker-public/templates/style-spec.md`
- `bundles/meeting-summary-worker-public/templates/rewrite-rules.md`
- `bundles/meeting-summary-worker-public/templates/scoring-rubric.md`
- `bundles/meeting-summary-worker-public/templates/production-prompt.md`
- `bundles/meeting-summary-worker-public/templates/patterns-detected.md`
- `bundles/meeting-summary-worker-public/templates/subagent-spec.md`
- `bundles/meeting-summary-worker-public/templates/io-contract.md`
- operator guides under `bundles/meeting-summary-worker-public/guides/`

Do **not** commit:
- `.env`
- bot tokens
- OAuth secrets
- raw runtime secret files

---

# Part 19 - Common mistakes to avoid

## Mistake 1 - Trying to fine-tune first
This slows you down and hides the style decisions.

## Mistake 2 - Using only positive examples
Negative examples are extremely useful because they show what the system should stop doing.

## Mistake 3 - Keeping the expertise only in chat history
If it is not in files, it is not durable.

## Mistake 4 - Using vague file names
Rigid naming dramatically reduces operational confusion.

## Mistake 5 - Treating persistent workers as guaranteed everywhere
Surface capabilities differ. Design for portability.

## Mistake 6 - Accepting generic AI next steps
Good next steps are concrete, attributable, and worth tracking.

---

# Part 20 - Minimal replication checklist

If another team wants to reproduce this quickly, the minimum checklist is:

## Dataset
- [ ] create Source / Target / Negative folders
- [ ] collect at least 5 paired examples
- [ ] use rigid naming conventions

## Distillation
- [ ] write style spec
- [ ] write rewrite rules
- [ ] write scoring rubric
- [ ] write production prompt

## Evaluation
- [ ] run at least 1 blind test
- [ ] compare against handwritten target
- [ ] update rules if failure modes repeat

## Operations
- [ ] store secrets in `.env`
- [ ] commit the style files to git
- [ ] define the command phrases operators should use
- [ ] choose the primary specialist surface

## Scaling
- [ ] publish operator guides
- [ ] define worker responsibilities
- [ ] add team access controls

---

# Final summary

The replicable recipe is:

1. **Collect examples**
2. **Distill the style into files**
3. **Use Source / Target / Negative structure**
4. **Validate with blind tests**
5. **Keep the expertise in version-controlled docs**
6. **Use Discord if you need true persistent thread-bound specialist workers**
7. **Treat the worker as an operator of the files, not the sole holder of the expertise**

That is the cleanest way to build a durable, high-signal meeting notes summariser in OpenClaw that a whole team can reproduce and extend.