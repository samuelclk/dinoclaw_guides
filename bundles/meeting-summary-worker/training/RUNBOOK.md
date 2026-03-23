# Meeting Summary Worker Runbook

## Standing Rule

Unless Stakesaurus explicitly says otherwise, any request to summarize meeting notes, transcripts, or Gemini-style meeting summaries must use the **Meeting Summary Worker** workflow.

This is the default path, not an optional nice-to-have.

## Goal

Produce **Samuel-style internal operator notes** rather than generic AI meeting minutes.

## Canonical source-of-truth files

Always load these before spawning the worker:
- `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/production-prompt.md`
- `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/style-spec.md`
- `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/rewrite-rules.md`
- `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/scoring-rubric.md`
- `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/subagent-spec.md`
- `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/io-contract.md`

## Manager responsibilities (main agent)

The main agent manages the worker end-to-end and should stay in **manager mode**, not do the summarization inline unless Stakesaurus explicitly asks otherwise.

Default workflow:
1. Identify the correct source file.
2. Load the canonical docs first.
3. Spawn the specialist summarization sub-agent.
4. Instruct it to use the canonical docs as source of truth.
5. Wait for completion.
6. Return only the final Samuel-style summary to Stakesaurus.
7. If the draft is obviously off-style, rerun with tighter instructions before replying.

Operational rule:
- Meeting-note summarization should be outsourced to the dedicated sub-agent by default so the main agent is not bogged down by production summarization work.
- The main agent owns routing, QA, retries, and user reporting.
- The sub-agent owns the actual summary generation.

## Invocation rule

Treat all of the following as triggers for this workflow:
- `msw <file>`
- `Use meeting summary worker: <file>`
- “summarize this meeting”
- “summarize these notes”
- “summarize this transcript”
- “use the meeting summary worker”
- “run the Samuel-style meeting summary sub-agent”

Preferred explicit triggers:
- `msw <file>`
- `Use meeting summary worker: <file>`

Default assumption: if the task is meeting-note summarization, use this runbook.

## Required worker instructions

The worker prompt should explicitly say:
- read the canonical files first
- use them as source of truth
- choose the correct Samuel-style output format
- compress hard
- preserve only durable facts
- remove generic AI/minutes phrasing
- do not invent action items
- output only the final summary

## Quality gate before replying

Before sending the result back, quickly check:
- Is the counterparty clear immediately?
- Did it preserve the real opportunity / blocker / dependency?
- Did it cut noise aggressively?
- Are next steps concrete and worth tracking?
- Does it sound like Samuel-style internal notes rather than Gemini / generic AI?

If no, rerun or tighten.

## Output expectation

Default user-facing output:
- only the final summary
- no commentary about the worker unless Stakesaurus asks
- no generic “here’s the summary” padding unless helpful in chat

## Override rule

If Stakesaurus explicitly asks for a different format, a fast rough summary, a comparison draft, a rubric score, or non-Samuel-style output, follow the explicit instruction instead of this default.
