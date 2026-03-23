# lidoclaw workspace docs

Shared export of the work-instance docs bundle.

## What’s here

### guides/
Operator-facing guides, setup docs, cheat sheets, and workflow references.

Good starting points:
- `guides/Meeting_Notes_Summariser_Index.md` — index for the meeting-summary system
- `guides/Discord_Setup_for_Persistent_Thread_Bound_Subagents_on_OpenClaw.md` — Discord lane setup, including manager-routed vs persistent ACP lane
- `guides/Runbook_Triggering_the_Meeting_Notes_Summariser.md` — invocation patterns
- `guides/Meeting_Transcript_Summary_Workflow.md` — transcript-to-summary workflow
- `guides/Operator_Guide_Building_a_High_Signal_Meeting_Notes_Summariser_in_OpenClaw.md` — longer operator guide

### meeting-summary-style-training/
Canonical meeting-summary worker materials.

Key files:
- `meeting-summary-style-training/RUNBOOK.md` — canonical workflow
- `meeting-summary-style-training/production-prompt.md`
- `meeting-summary-style-training/style-spec.md`
- `meeting-summary-style-training/rewrite-rules.md`
- `meeting-summary-style-training/scoring-rubric.md`
- `meeting-summary-style-training/subagent-spec.md`
- `meeting-summary-style-training/io-contract.md`

## Key operational lesson

For the Discord Meeting Summary Worker lane:
- use a normal **manager-routed** binding: `type: "route"`
- do **not** use a direct `type: "acp"` binding if the goal is:
  - runbook/doc loading first
  - manager-mode orchestration
  - `sessions_spawn(runtime="subagent")`
  - final manager-delivered Samuel-style summary

## Proven `msw` flow

`msw <file>` on Discord should follow this path:
1. Discord message lands in the routed manager session
2. manager loads `meeting-summary-style-training/RUNBOOK.md`
3. manager loads canonical docs
4. manager locates the source file
5. manager calls `sessions_spawn(runtime="subagent", ...)`
6. child worker completes
7. manager returns the final summary

## Trigger phrases

Preferred triggers:
- `msw <file>`
- `Use meeting summary worker: <file>`

## Notes

This folder is a staged copy from:
- `/home/lidoclaw/.openclaw/workspace`

It is intended as a shared reference bundle, not the live source of truth.
