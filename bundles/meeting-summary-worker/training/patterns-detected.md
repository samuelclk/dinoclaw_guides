# Patterns Detected from the First 10 Training Examples

## High-confidence patterns

### 1. Samuel rewrites for retrieval, not for publication
The target summaries are optimized for quickly remembering:
- what the meeting was about
- what mattered
- what to do next

They are not optimized to be a standalone polished artifact for outsiders.

### 2. Business implication beats transcript fidelity
Samuel consistently removes wording that is merely descriptive and keeps wording that captures:
- commercial angle
- integration angle
- blocker
- timing
- next owner

### 3. “Key discussion point” means “surviving takeaway”
The targets do not mirror all topics discussed.
A point survives only when it materially affects future action or understanding.

### 4. Negative examples are too verbose by a large margin
Approximate character lengths:
- targets: ~700 to ~1,700 chars
- negatives: ~6,000 to ~9,000 chars

This is not a minor style difference. It is a different objective.
Samuel’s style is severe compression.

### 5. Samuel often preserves these categories
Most target summaries preserve some mix of:
- partner context / role
- specific opportunity angle
- blocker / dependency
- commercially relevant quantity or timing
- next steps

## Medium-confidence patterns

### 6. Two-format preference appears intentional
- `Call with ...` for tighter 1:1 or simpler conversations
- `Meeting with ...` + participants/notes/next steps for more formal or multi-party records

### 7. Numbers are kept selectively
Numbers survive when they anchor:
- opportunity size
- timeline
- yield / rate context
- commercial relevance

### 8. Samuel likes named bundles of opportunity angles
Examples include:
- `5 angles to explore`
- `2 low-hanging fruits`
- `Discussed 3 angles`

This is a strong pattern for making broad discussion concrete.

### 9. Samuel will compress product explanations into shorthand
He assumes the reader already understands terms like:
- stETH
- Lido Earn
- DI
- Morpho
- stVaults
- fee share

So he removes beginner-facing explanation unless it changes the commercial implication.

## Lower-confidence / needs more raw examples

### 10. Raw transcript extraction style is only partially learned
Only examples 1–3 show transcript -> target conversion directly.
That is enough to infer some rules, but not enough to fully trust subtle inclusion/exclusion judgment on raw transcripts yet.

## Early operating conclusion

The immediate system should be optimized as:

- **rewrite engine first** for Gemini -> Samuel-style summaries
- **raw transcript extraction engine second**, improved as more transcript/target pairs are added

That matches the current dataset shape and should produce the best near-term quality.

## Update after explicit Gemini->target pass on examples 1-3

That extra pass confirmed three things:
- the first 3 examples also contain strong **Gemini-specific anti-pattern** signal, not just transcript-extraction signal
- Samuel's style consistently preserves **pivots, blocker diagnoses, and dependency logic** over background or process detail
- the rewrite side of the dataset is now effectively reinforced across **all 10 examples**, while raw transcript extraction is still only supported by the first 3
