# Rewrite Rules: Raw / AI Notes -> House-Style Summary

This file turns the example set into actionable transformation rules.

## Primary rewrite instruction

Given a raw transcript or Gemini meeting summary, rewrite it into a **compressed internal summary** that preserves only:
- counterparties
- meaningful strategic / operational points
- blockers / dependencies
- concrete next steps

## Step-by-step transformation

### 1. Identify the meeting type

Classify source into one of:
- **simple call** -> use compact `Call with ...` structure
- **multi-party / strategic meeting** -> use `Meeting with ...` + `Participants / Notes / Next Steps`

Heuristic:
- 1–2 external participants + straightforward BD discussion -> compact call format
- multiple orgs / more formal meeting / more archival value -> structured meeting format

### 2. Extract the durable facts

Capture only facts likely to matter after the meeting is over:
- who the counterpart is and why they matter
- what product / partnership angle was discussed
- what constraint or blocker emerged
- what concrete opportunity emerged
- what specific follow-up is needed

### 3. Delete meeting noise aggressively

Delete:
- introductions
- pleasantries
- audio / lag / scheduling details
- long product explanations
- transcript timestamps
- repeated paraphrases of the same point
- generic summaries that merely restate the meeting topic

### 4. Reframe from explanation -> implication

Convert from:
- “the operator explained how X works using A, B, and C”

Into:
- “Presented <org> Earn / DI integration model”
- “Explained that Mellow is the relevant curator for RWA allocation discussions”

Rule: preserve the consequence, not the lecture.

### 5. Collapse into business angles

When many details point to the same opportunity, collapse them into one angle.

Examples:
- multiple sentences about collateral, trading, and exchange mechanics ->
  `Enabling stETH as trading collateral on Swarm`
- long explanation of JPY stablecoin + borrowing mechanics ->
  `High interest in using stETH as collateral to borrow the new JPY stablecoin via Morpho`

### 6. Keep useful numbers, drop decorative ones

Keep numbers only if they affect:
- priority
- scale
- feasibility
- timing
- economics

Keep examples like:
- `~5,000 ETH`
- `3–5 months`
- `9–11% borrowing rates`
- `June 2026`

Drop numbers if they are only explanatory texture.

### 7. Write notes in the operator’s preferred density

Preferred note line styles:
- direct declarative sentence
- one line per important takeaway
- nested bullets only when substructure adds clarity

Examples:
- `<example> already utilizes <org> Finance to deploy their Ethereum users' assets for their "simple earn" product.`
- `<example> Exchange and OTC do not support margin trading yet.`
- `Constance Chen offered to help expedite stETH listing internally.`

### 8. Rewrite next steps into real operator actions

For each candidate next step, ask:
- who owns this?
- is this a real deliverable / intro / check / revisit?
- would the operator actually track this later?

If yes, keep and sharpen.
If not, delete.

Preferred syntax:
- `the operator to ...`
- `<Counterparty> to ...`
- `Share ...`
- `Get confirmation on ...`
- `Revisit discussion ...`

## Explicit Gemini -> target insights from early examples

These examples had raw transcripts, but after the explicit Gemini-source pass they now also teach several direct rewrite rules.

### Preserve the pivot, not the setup
If the meeting's value lies in the operator redirecting the conversation from one path to a better one, preserve that pivot explicitly.

Example pattern:
- Gemini: broad recap of company background + explored ideas
- house-style target: `Explained why the original ask should go elsewhere, then redirected to 5 angles to explore`

### Prefer dependency logic over meeting logistics
For GTM / partnership discussions, the operator keeps the key dependency that determines whether outreach is worth doing.

Example pattern:
- not all event chatter
- but `this only makes sense if <example> and <org> has an active integration`

### Rewrite low-demand language into blocker diagnosis
When Gemini says adoption is low, rewrite into the actual reasons if the source supports it.

Example pattern:
- Gemini: low client uptake of staking
- house-style target: `The problem seems to be two-fold...`

### Downgrade housekeeping asks unless they change progress
Gemini often promotes requests like one-pagers, extra intros, blocker-name gathering, or procedural follow-ups into major next steps.
the operator keeps them only if they materially affect progress.

## Negative -> target conversion rules

### Anti-pattern 1: Gemini over-explains known products

Negative:
- long paragraphs on how <org> Earn, curators, looping, or interfaces work

Target behavior:
- compress to one line unless the mechanic itself is the point

### Anti-pattern 2: Gemini preserves chronology

Negative:
- meeting recap unfolds in transcript order

Target behavior:
- reorder into strategic importance, not chronology

### Anti-pattern 3: Gemini treats all discussed ideas equally

Negative:
- every thread gets equal narrative weight

Target behavior:
- rank and retain only the 3–7 points that matter

### Anti-pattern 4: Gemini inflates next steps
n
Negative:
- broad, generic, or inferred actions presented as commitments

Target behavior:
- keep only actions clearly implied or stated, rewritten more crisply if needed

### Anti-pattern 5: Gemini includes social / procedural clutter

Negative:
- technical difficulties, who rejoined, where people are based, etc.

Target behavior:
- remove unless directly relevant to execution or geography strategy

## Raw transcript -> target extraction rules

These are inferred mostly from examples 1–3.

### Rule A: skip most of the explanatory middle
When a transcript contains long product explanation sections, keep only:
- the actual ask
- the key clarification
- the redirected angle
- the resulting next steps

### Rule B: summarize multiple explored ideas as named angles
If a discussion surfaces several collaboration possibilities, rewrite as:
- `3 angles`
- `5 angles to explore`
- a short list of named opportunity vectors

### Rule C: prefer blocker diagnosis over generic “low demand” wording
Example pattern:
- not just `no demand`
- but `problem seems two-fold: sales team not knowing how to sell liquid staking; limited trading integration`

### Rule D: preserve strategic redirection
If the operator redirects the conversation from an unworkable path to a better one, keep that pivot.
This seems central to how he interprets meetings.

## Output checklist

Before finalizing, verify:
- Does the first line identify the counterpart clearly?
- Did I remove all greetings / transcript junk?
- Are the notes organized by importance, not sequence?
- Did I keep only points with operational value?
- Are next steps concrete and owned where possible?
- Does this sound like internal notes, not auto-generated meeting minutes?

## Suggested generation template

### Compact call template

`Call with <name> (<role>):`
- point 1
- point 2
- point 3
- point 4
`Next steps:`
- owner/action
- owner/action

### Structured meeting template

`Meeting with <org> & <org>`
`Participants:`
- participant list

`Notes`
- key note 1
- key note 2
- key note 3

`Next Steps`
- action 1
- action 2
- action 3

## Practical recommendation for future runs

For best quality, run the workflow in two passes:

1. **Extraction pass**
   - from transcript / Gemini notes -> candidate facts, decisions, next steps
2. **house-style rewrite pass**
   - compress candidate facts into this house style

This should outperform a single monolithic prompt, especially when the source is a raw transcript.