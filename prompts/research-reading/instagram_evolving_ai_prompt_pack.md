# Copy-Paste Prompt Pack

Use these prompts after uploading a paper set to your AI tool of choice.

---

## Prompt 1 -- Intake Protocol

I'm going to share [NUMBER] papers on [TOPIC]. Before I ask any questions, please do the following:

1. List every paper in a table with columns:
   - Author(s)
   - Year
   - Core Claim (one sentence, <=20 words)

   If a paper has no explicit thesis, infer the central argument from its conclusions.

2. Group the papers into 2-5 clusters based on shared theoretical assumptions or frameworks. Name each cluster and briefly explain (1-2 sentences) what unites the papers within it.

3. Flag any direct contradictions between papers -- where two or more authors make mutually exclusive claims about the same phenomenon. List each as:
   - `Paper A vs. Paper B -- contested claim`

Do not summarize each paper individually. Focus only on the three tasks above.

---

## Prompt 2 -- Contradiction Finder

Across all uploaded papers, identify the most significant points where two or more authors make claims that directly contradict each other.

Only include genuine contradictions -- mutually exclusive claims on the same issue.

Exclude cases of mere difference in emphasis or scope.

Present your findings as a table with the following columns:
- Contested Claim
- Position A (Paper, Year)
- Position B (Paper, Year)
- Root Cause of Disagreement

For Root Cause, choose from:
- methodology
- dataset
- time period
- definition of terms
- other (explain)

Aim for 5-10 contradictions. If fewer exist, list all you find.

---

## Prompt 3 -- Citation Chain

From the uploaded papers, identify the 3 concepts that appear most frequently across multiple papers (referenced by name, debated, or built upon).

For each concept, trace its intellectual history using only the evidence in the uploaded papers:

**Concept Name:**
- **Origin:** Who first introduced or defined it (within this set)?
- **Challenge:** Which paper(s) questioned or challenged it, and how?
- **Refinement:** Which paper(s) modified or extended it, and how?
- **Current Status:** Settled, contested, or still evolving -- based on this literature?

Present each concept as a structured outline. If a concept lacks a clear challenger or refinement in these papers, state that explicitly rather than guessing.

---

## Prompt 4 -- Gap Scanner

Based only on the uploaded papers, identify the 5 most significant research gaps that these papers collectively acknowledge, imply, or fail to address.

For each gap:
- **Gap:** [State the unanswered question clearly in 1-2 sentences]
- **Why it exists:** Choose from:
  - methodological barrier
  - lack of data
  - topic too niche
  - assumed but untested
  - ethical/logistical constraint

  Explain briefly.

- **Closest paper:** Which uploaded paper came closest to addressing it, and where did it fall short?
- **Path to resolution:** What would be needed to close this gap (methodology, data, resources, etc.)?

Rank the 5 gaps from most to least significant, and briefly explain your ranking criterion (e.g. theoretical importance, practical impact, feasibility of resolution).

If fewer than 5 genuine gaps exist, list all you can identify and explain why the set is limited.

---

## Prompt 5 -- Methodology Audit

Compare the research methodologies used across all uploaded papers.

### Step 1 -- Classification Table

Create a table:
- Paper (Author, Year)
- Methodology Type
- Data Source
- Sample Size (if stated)
- Key Limitation Noted by Authors

Use the methodology type that best fits each paper. Don't force papers into the categories below -- add new categories as needed.

Suggested types:
- Survey
- Experiment (RCT)
- Quasi-experiment
- Simulation
- Meta-analysis
- Case study
- Computational/ML
- Literature review
- Ethnography
- Secondary data analysis

### Step 2 -- Synthesis

- Which methodology type appears most frequently? Suggest why based on the papers' stated rationale.
- Which methodology is absent or rare despite being relevant to the research questions?

### Step 3 -- Weakest Methodology

Identify the paper whose methodology is most vulnerable to criticism. Evaluate using these criteria:
- sample size adequacy
- control for confounds
- replicability
- transparency of reporting

State which criterion it fails most clearly.

---

## Prompt 6 -- Master Synthesis

Using the uploaded papers as your only source, write a synthesis of this body of literature. **Do NOT summarize individual papers.** Instead, write across the entire literature:

1. **Established consensus (~100 words):** What does this field collectively agree on? Cite at least 2 papers that support each claim you make here.

2. **Active debates (~100 words):** What do researchers in this field meaningfully disagree about? Name the disagreeing positions without naming individual papers.

3. **Strongest evidence (~100 words):** What claims in this literature are supported by the most consistent, replicated, or methodologically robust evidence?

4. **The key open question (~80 words):** End with the single most important unanswered question in this field -- the one whose resolution would most change the others.

**Total:** 400 words maximum.

No hedging phrases like "it seems" or "some argue." State clearly.

If the papers lack sufficient consensus to populate a section, say so explicitly.

---

## Prompt 7 -- Assumption Killer

From the uploaded papers, identify the 5-8 most consequential assumptions that the majority of these papers share but never explicitly test, justify, or acknowledge as assumptions.

Focus on assumptions that are:
- foundational to the conclusions drawn, and
- plausibly false or context-dependent.

For each assumption:
- **Assumption:** [State it as a declarative claim, e.g., "X causes Y under all conditions"]
- **Shared by:** Name 2-3 papers that rely on it most heavily.
- **Risk level:** Rate as Low / Medium / High based on how much of the literature would be undermined if the assumption is false.
- **Consequence:** Explain what would change -- would conclusions need revision (low impact), key findings be invalidated (medium), or the entire research paradigm collapse (high)?

Rank assumptions from most to least consequential.

---

## Prompt 8 -- Knowledge Map Builder

Based only on the uploaded papers, create a structured knowledge map of this literature. Present it as a clean outline (no prose paragraphs).

### KNOWLEDGE MAP

1. **Central Claim:** The single proposition that most of this field's work tries to support, challenge, or refine. If no single claim unifies the field, name 2 competing centres instead.

2. **Supporting Pillars (3-5):** Well-established sub-claims with strong evidentiary support across multiple papers. For each:
   - `[Claim]` -- supported by: `[Paper 1]`, `[Paper 2]`

3. **Contested Zones (2-3):** Areas of genuine, active disagreement. For each:
   - `[Issue]` -- `[Position A]` vs. `[Position B]`

4. **Frontier Questions (1-2):** Questions this literature raises but cannot yet answer. State as explicit questions.

5. **Newcomer Reading List (3 papers):** For each paper, state:
   - `[Author, Year]` -- why a newcomer should read this first.

**Selection criterion:** foundational to understanding the field, not just most cited.

---

## Prompt 9 -- The 'So What' Test

Summarize this entire body of research for a smart non-expert who has never read any of it. Respond in exactly three numbered points. Each point should be 2-3 sentences maximum.

Write as if speaking to an intelligent person with no domain knowledge.

1. **What has been proven:** The strongest, most reliable finding from this literature -- stated as a direct claim with no hedging. No "suggests" or "may indicate."

2. **What is still unknown:** The most significant thing this field has not yet figured out -- stated honestly, without minimizing the uncertainty.

3. **Why it matters:** The single most important real-world implication. If no direct application exists, state the biggest theoretical consequence instead.

**Rules:**
- No jargon.
- No citations.
- No qualifications that weaken the core point.

If you cannot make a statement confidently based on the papers, say so -- don't fabricate certainty.

---

## Suggested Workflow

If you're using all 9 prompts in sequence, this order works best:
1. Intake Protocol
2. Contradiction Finder
3. Citation Chain
4. Gap Scanner
5. Methodology Audit
6. Master Synthesis
7. Assumption Killer
8. Knowledge Map Builder
9. The 'So What' Test
