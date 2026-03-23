# House-Style Meeting Summary Spec

Built from a set of paired examples and anti-pattern examples.

## Core objective

Produce a **high-signal operator summary**, not a transcript recap.

The summary should help the operator quickly remember:
- who the counterparty is
- what actually matters strategically or operationally
- what the real opportunities / constraints are
- what concrete follow-ups should happen next

## What the target style optimizes for

1. **Compression over completeness**
   - Keep only the points that change follow-up, prioritization, or understanding.
2. **Operational usefulness over chronology**
   - Organize by substance, not by meeting sequence.
3. **Concrete business meaning over literal transcript fidelity**
   - Rewrite vague discussion into crisp commercial / integration language.
4. **Decision support over meeting theater**
   - Ignore small talk, intros, technical glitches, and filler.
5. **Actionability**
   - End with explicit next steps, preferably with owners.

## Canonical output shapes

There are **two valid output formats** in the examples.

### Format A — Short call summary
Used for simpler 1:1 or lighter business-development calls.

Pattern:
- Header: `Call with <name> (<role>):` or `Call with <name> and <name>:`
- 4–8 concise bullets / numbered points
- `Next steps:` section at the end

Common traits:
- No participant block
- Minimal formatting
- Dense factual statements
- May use numbered hierarchy when there are subpoints

### Format B — Structured meeting note
Used when the meeting has multiple participants, more context, or a partnership workflow worth preserving.

Pattern:
- Header: `Meeting with <org> & <org>`
- `Participants:` block
- divider line
- `Notes`
- divider line
- `Next Steps`

Common traits:
- Bullet-driven
- Slightly more polished and archival
- Better for multi-party or more strategic conversations

## Tone and voice

- Direct
- No fluff
- No salesy adjectives unless they matter
- No narrative scene-setting
- No unnecessary hedge words
- No fake precision
- Sounds like internal working notes written by someone who already knows the space

## Inclusion rules: what becomes a main note

Include items that affect one or more of the following:
- partnership angle
- integration feasibility
- distribution opportunity
- product fit
- regulatory constraint
- custody / collateral / infrastructure constraint
- timing that changes sequencing
- internal escalation needed
- commercial leverage (fees, TVL, listing support, co-marketing, etc.)

Good candidates:
- actual interest in the relevant product, integration path, or commercial offering
- specific use cases (collateral, distribution, treasury, custody, structured products)
- constraints that block adoption
- internal champions / relevant owners
- quantified context when it helps prioritization (e.g. ETH held, TVL, timeline, rates)
- 2–5 clear collaboration angles

## Exclusion rules: what should usually be omitted

Do **not** include unless genuinely decision-relevant:
- greetings / intros
- audio issues
- travel / location chatter
- chronology of who said what when
- repeated explanations of how <org> works
- overlong product education paragraphs
- restating every exploratory idea discussed
- speculative detail that did not survive into a clear takeaway
- AI-style over-explanation of mechanics if one line will do

## How to handle key discussion points

A “key discussion point” in house-style output is **not** just something that was discussed.
It is a point that passes at least one of these tests:

- creates a concrete opportunity
- identifies a blocker / dependency
- reframes the opportunity into a more useful angle
- changes who needs to be involved next
- reveals commercial or regulatory constraints
- provides useful sizing / timing

Preferred form:
- short declarative sentence
- one idea per line
- preserve the business implication, not the entire backstory

## How to handle next steps

Only include next steps that are:
- real
- specific
- attributable
- useful for follow-through

Preferred style:
- owner + action
- concrete deliverable / intro / check / revisit
- omit fake next steps like “continue exploring” unless that is genuinely the only outcome

Good:
- `owner to share the relevant materials`
- `partner contact to help unblock the listing or integration path`
- `Team to confirm integration requirements and timelines`

Weaker / avoid unless unavoidable:
- `Continue discussing opportunities`
- `Explore synergies further`

## Compression rules

When reducing transcript or AI-generated notes into target style:
- collapse repeated points into one line
- merge related subpoints under a single commercial angle
- keep numbers only when they help prioritization or constraint analysis
- remove explanatory scaffolding around known concepts
- prefer one line with implication over three lines of setup

## Role of numbers / specifics

Keep numbers when they matter for:
- size of opportunity
- urgency
- feasibility
- regulatory timing
- commercial leverage

Examples from the dataset:
- ETH balances / staked amounts
- launch windows
- borrowing rates
- TVL thresholds
- expected approval timing

Drop numbers that are decorative or not actionable.

## Relationship to source type

### When source is a raw transcript
The summary should:
- infer the few durable takeaways
- strip all meeting noise
- avoid transcript order
- preserve only strategic and operational memory

### When source is a AI summary
The task is mostly a **rewrite-and-prune** pass:
- remove AI’s narrative structure
- remove detail bloat
- strip generic explanations
- keep only the points the operator would likely refer back to later

## Common anti-patterns seen in negative examples

1. **Chronology bloat**
   - too much “first they said / then he replied” structure
2. **Explanatory overkill**
   - multiple paragraphs explaining product mechanics the operator already knows
3. **False importance**
   - details that sound substantial but do not change next actions
4. **Weak next steps**
   - generic or inflated action items
5. **Meeting-theater noise**
   - intros, glitches, context that is socially real but operationally useless

## First-pass inferred preferences

- the operator likes summaries that read like **internal operator notes**, not meeting minutes.
- The operator prefers **compressed business framing** over broad descriptive prose.
- He often rewrites toward:
  - opportunity framing
  - blocker framing
  - partner ask / owner clarity
  - clear next action
- He is comfortable with domain shorthand and does not need beginner-friendly explanations.

## Confidence notes

- Strong confidence on **rewrite style** from the available examples.
- After the explicit AI->target pass on early examples, the rewrite signal is stronger because those first 3 examples now contribute to rewrite learning as well, not just transcript extraction contrast.
- Moderate confidence on **raw transcript extraction style** because only examples 1–3 include raw transcripts.
- Likely strongest early performance when rewriting AI summaries; more raw transcript examples would improve extraction judgment.