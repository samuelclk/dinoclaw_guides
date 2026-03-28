# Output Template

Use this as the default concise output and local archive format.

## Chat reply template

```md
**Summary**

1. First key point.

2. Second key point.

3. Third key point.

**Fact-check**

1. First fact-check point.

2. Second fact-check point.

3. Third fact-check point.
```

Fact-check bullets should be based on externally checked claims where possible, not only on the source transcript.

## Local archive template

```md
# human-readable-topic

- Title: Human readable title
- Source: https://example.com/...
- Artifact dir: `tmp/...`

**Summary**

1. First key point.

2. Second key point.

**Fact-check**

1. First fact-check point.

2. Second fact-check point.
```

## Naming guidance

Prefer names like:
- `ai-expert-personas-hurt-accuracy.md`
- `four-cli-tools-for-claude-code.md`
- `obsidian-second-brain-for-claude-code.md`

Avoid names like:
- `DWZD8ajDqT8.md`
- `run_12345.md`

Use opaque IDs only when no better topic slug can be derived.
