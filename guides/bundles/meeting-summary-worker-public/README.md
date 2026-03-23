# Meeting Summary Worker (Public Bundle)

This bundle contains the public-safe, reusable parts of a meeting-summary workflow for OpenClaw.

## Included

- `guides/` — operator-facing docs, quickstarts, runbooks, and setup guides
- `templates/` — genericized prompt/spec/rubric/worker-contract files

## Excluded on purpose

- real training examples
- blind tests
- outputs
- private counterparties / org names
- Drive IDs / private URLs
- environment-specific personal paths where avoidable

## Intended use

Use this bundle as a starting point for building your own house-style meeting summariser:
1. create your own source/target/negative dataset
2. adapt the templates to your team style
3. run blind tests
4. refine the rules over time
