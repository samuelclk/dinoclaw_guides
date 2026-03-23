# Meeting Summary Worker: Inputs / Outputs Contract

## Inputs

### Primary input types

1. **Raw transcript**
   - source: shared storage `Source` folder or local mirrored file
   - examples: exported transcript text, meeting transcript doc text

2. **AI summary / notes**
   - source: shared storage `Source` folder or local mirrored file
   - used either as production input or blind-test source

3. **Negative example** (optional but useful)
   - source: shared storage `Negative` folder or local mirrored file
   - purpose: clarify what style to avoid or compare against

4. **Target summary** (optional for evaluation only)
   - source: shared storage `Target` folder or local mirrored file
   - purpose: review / scoring / gap analysis after draft generation

### Required reference docs

- `bundles/meeting-summary-worker-public/templates/production-prompt.md`
- `bundles/meeting-summary-worker-public/templates/style-spec.md`
- `bundles/meeting-summary-worker-public/templates/rewrite-rules.md`
- `bundles/meeting-summary-worker-public/templates/scoring-rubric.md`
- `bundles/meeting-summary-worker-public/templates/subagent-spec.md`

## Naming conventions

### Training examples
- `training_source_<n>_<slug>`
- `training_target_<n>_<slug>`
- `training_negative_example_<n>_<slug>`

### Blind tests
- `blindtest_source_<n>_<slug>`
- `blindtest_negative_example_<n>_<slug>`
- later, if needed: `blindtest_target_<n>_<slug>`

## Outputs

### Output A: Production summary
A house-style summary in one of two formats:
- compact `Call with ...` format
- structured `Meeting with ...` format

### Output B: Self-score (optional)
A rubric-based self-score using:
- counterparty clarity
- signal selection
- business framing
- compression quality
- next-step quality
- tone match

### Output C: Review package (optional)
If target exists:
- draft summary
- comparison to target
- score / gap analysis
- possible rule update suggestions

### Output D: Saved artifact
Worker should save durable outputs under a local path such as:
- `bundles/meeting-summary-worker-public/templates/output/`
- `bundles/meeting-summary-worker-public/templates/blindtests/<case>/`

## Default workflow

1. Load canonical docs
2. Read source input
3. Read negative example if available
4. Generate house-style summary
5. Optionally self-score
6. Save result locally
7. If approved / useful, commit and push, then return GitHub URL

## Surface independence

This contract is designed to work across:
- chat surface
- Discord
- other chat surfaces
- manual Drive-drop workflows

The surface should change only the routing layer, not the summarization logic.