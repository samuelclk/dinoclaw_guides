---
name: research-reading
description: Break down complex research papers, whitepapers, RFCs, standards, technical specs, and dense technical writing into decision-useful summaries. Use when the request involves deep reading, extracting claims/assumptions/methods/limitations, comparing multiple papers, evaluating evidence quality, translating jargon-heavy material into plain English, or producing structured research syntheses for practitioner, expert, or ELI5 audiences.
---

# Research Reading

Use this skill for deep-reading tasks where a normal quick answer is not enough.

## Route or Answer Directly

Use this skill when one or more of these are true:
- the source is a paper, whitepaper, RFC, spec, standard, or long technical article
- multiple documents need comparison
- the user asks for assumptions, methods, limitations, evidence quality, or open questions
- the user wants ELI5, practitioner, or expert translation of dense material
- the source is jargon-heavy, mathematical, legalistic, or domain-specific

Answer directly instead when the request is only:
- a quick term definition
- a short factual clarification
- a lightweight follow-up from already-produced analysis

For routing guidance, read `references/routing.md`.

## Default Workflow

1. Identify the user's real goal.
   - summary
   - critique
   - comparison
   - explanation
   - implications / recommendations

2. Identify the audience.
   - ELI5
   - practitioner
   - expert

3. Read the material and extract:
   - core claim or thesis
   - problem being solved
   - mechanism / argument structure
   - evidence or methodology
   - assumptions
   - limitations and risks
   - practical implications

4. Separate clearly:
   - what the source says
   - what the evidence supports
   - what is speculative
   - what matters in practice

5. Return a structured output, not a wall of prose.

For the full output pattern, read `references/output-format.md`.

## Interpretation Rules

- Do not paraphrase the abstract and call it analysis.
- Do not over-trust the author's framing.
- Separate novelty from marketing.
- Separate mechanism from evidence.
- Call out weak methodology, vague claims, benchmark gaming, and unstated assumptions.
- If the source is weak or ambiguous, say so directly.

## Comparison Mode

When comparing documents, include:
- where they agree
- where they disagree
- which is more rigorous
- which is more practically useful
- what difference actually matters for a decision-maker

## Style

- Prefer bullets and short sections.
- Be concise but precise.
- Translate jargon into plain English.
- Default to practitioner usefulness unless the user asks otherwise.
- Avoid fluff, filler, and fake certainty.
