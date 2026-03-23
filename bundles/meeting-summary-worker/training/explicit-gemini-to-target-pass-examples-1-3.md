# Explicit Gemini -> Target Pass for Examples 1-3

This pass treats the Gemini summaries for examples 1-3 as **source inputs** and compares them directly against Samuel's handwritten targets.

## Why this pass matters

The earlier analysis used examples 1-3 mainly as:
- raw transcript -> target
- with Gemini as a contrastive negative

This document adds the missing explicit lens:
- Gemini summary -> Samuel-style target

That helps isolate **Gemini-specific rewrite failures** even in the examples that also had raw transcripts.

---

## Example 1 — Inveniam

### What Gemini overemphasized
- long explanation of how Lido Earn works
- detailed background on Inveniam's structure, acquisition history, geography, and asset sourcing
- expanded explanation of Swarm mechanics
- procedural next step around a one-pager
- meeting noise like audio issues and chronology

### What Samuel kept instead
- the core gating clarification: Mellow is the curator, so RWA allocation discussion belongs there
- the strategic redirection away from co-curation into **5 concrete collaboration angles**
- one crisp next step: Jackson/Peter to review the 5 angles internally

### Gemini-specific failure modes exposed
1. **Background gravity**
   - Gemini gets pulled into describing the counterparty's business in too much detail.
2. **Mechanics drift**
   - Gemini spends too much space explaining how Lido Earn / Swarm work.
3. **Process over action**
   - Gemini elevates the one-pager ask; Samuel treats it as secondary versus the 5 collaboration angles.
4. **Missed conversational pivot**
   - Samuel's target preserves the key move: redirecting an unworkable path into better collaboration options.

### Rule learned
When Gemini describes a rich company background plus many details, compress toward:
- the gating clarification
- the named opportunity set
- the single real next decision/action

---

## Example 2 — Kraken

### What Gemini overemphasized
- relocation / coworking chatter
- event format detail and travel planning texture
- long narrative around APAC and institutional event strategy
- too many next steps, some of which are intermediate conversational tasks

### What Samuel kept instead
- Kraken is prioritizing custody first, with Earn products in 3-5 months
- possibility of Lido integration onto Kraken Earn
- AU/SG institutional market focus
- side events / roadshows as GTM vector
- 2 concrete low-hanging-fruit targets: Bowman and Ark
- the dependency that outreach only makes sense if there is active Lido/Kraken integration
- 3 crisp next steps

### Gemini-specific failure modes exposed
1. **Social-detail bleed**
   - Gemini carries context that is humanly real but operationally useless.
2. **Over-expansion of GTM planning**
   - Many event-format observations are compressed by Samuel into a single practical line.
3. **Intermediate-task inflation**
   - Gemini includes steps like getting blocker names or arranging travel; Samuel retains the actions that matter strategically.
4. **Under-compression of dependency logic**
   - Samuel sharply preserves the key qualifier: GTM collaboration only works if product integration is active.

### Rule learned
For BD / GTM meetings, prefer:
- market focus
- partnership dependency
- low-hanging-fruit targets
- a short action list

Delete lifestyle, travel, or event-format chatter unless it directly changes execution.

---

## Example 3 — Ceffu

### What Gemini overemphasized
- travel recap and role introductions
- broad narrative about staking uptake problems
- detailed explanation of Mirror X / Mirror Reserve models
- too many suggested next steps and collateral mechanics details

### What Samuel kept instead
- new POC
- no current staking demand / no inflows to Lido
- sharper diagnosis: two-fold problem
  - sales team cannot sell liquid staking
  - trading integration is limited
- potential unlock: stETH as trading collateral on Binance via Ceffu's off-exchange settlement path
- condition: needs a sufficiently large client request
- 2 pragmatic next steps: work through QRT, extend lunch-and-learn

### Gemini-specific failure modes exposed
1. **Model-detail overretention**
   - Gemini preserves too much infrastructure explanation.
2. **Weak problem framing**
   - Gemini says uptake is low; Samuel diagnoses why.
3. **Too many next steps**
   - Samuel reduces to the highest-leverage follow-ups.
4. **Less useful naming**
   - Gemini keeps SEU/Mirror product detail; Samuel rewrites into the specific operational unlock that matters.

### Rule learned
When a Gemini summary reports low demand, rewrite toward:
- root-cause diagnosis
- specific unlock condition
- only the highest-leverage follow-ups

---

## New rewrite rules learned from this explicit pass

### 1. Preserve the pivot, not the setup
If the value of the meeting lies in a redirect (from one angle to a better angle), keep the pivot explicitly.

### 2. Reduce company background to only what changes the opportunity
Counterparty backstory should survive only if it affects fit, timing, or go-to-market.

### 3. Prune intermediate next steps
Gemini often promotes conversational housekeeping into action items. Samuel keeps only the follow-ups that change progress.

### 4. Convert “low demand” into a diagnosis when evidence exists
If the source contains evidence of why adoption is weak, rewrite into the blocker structure rather than keeping generic demand language.

### 5. Collapse operating detail into one commercial implication
If Gemini spends multiple paragraphs on mechanism, extract the one implication Samuel would want to remember.

## Net effect on the overall training set

After this pass:
- Examples 1-3 now contribute both to **raw transcript extraction learning** and **Gemini -> target rewrite learning**.
- This strengthens the Gemini-rewrite side of the dataset from 7 examples to effectively 10 examples.
- The raw-transcript extraction side remains 3 examples.
