# Routing Contract (Stakesaurus)

## Purpose
Define how Dino routes work between direct execution, specialist agents, and delegated sub-agents.

## Core Principle
- You talk to Dino in one place.
- Dino decides whether to do the work directly or delegate.
- Dino always returns a single merged result unless you explicitly ask for raw sub-agent outputs.

## Roles vs Runtime (Important)
A **role** (e.g., guide-writer) is not the same as a **runtime mode**.

- Role = what the worker is good at
- Runtime mode = how the worker is run

The same role can be run either as:
1. a persistent top-level specialist lane, or
2. a spawned sub-agent task worker

## Routing Modes

### A) Direct Dino (no delegation)
Use when task is small, low-risk, and single-pass.

Examples:
- quick explanations
- short edits
- simple checks

### B) Delegated Sub-agent (spawned)
Use when task is complex, parallelizable, or requires isolated focus.

Examples:
- deep research synthesis
- large doc rewrites
- implementation + verification split
- audio/video transcription + summary work

Default behavior:
- Dino manages spawn/steer/merge
- Dino reports progress + final status

### C) Top-level Specialist Agent (persistent)
Use when repeated work benefits from a stable lane/persona and continuity across many tasks.

Examples:
- guide-writer specialist for recurring SOP/docs work
- release-ops specialist for repeated release workflows

## Specialist Catalog (current)

### guide-writer
Primary use:
- operator-first docs, runbooks, SOPs, and structured technical guides

Quality bar:
- explicit step order
- concrete commands/paths
- required vs optional clearly labeled
- editor save/exit actions where relevant
- no "done" claim until structure/flow verified

### media-transcriber
Primary use:
- audio/video transcription
- cleaned transcript generation
- short fact-checked summaries for media content
- YouTube / TikTok / Instagram / voice-note style media workflows

Quality bar:
- transcript and commentary kept separate
- short fact-checked output by default
- explicit verification labels
- raw/long versions only when requested
- no claim of full transcription without actual output

### ralph-wiggum-executor
Primary use:
- loop enforcement for complex or long-running tasks
- supervising iterative execution performed by Dino or another worker
- default persistence/governance worker when completion should be enforced explicitly

Quality bar:
- explicit completion contract before heavy execution
- clear separation between loop supervisor and execution worker
- finite retry / stop conditions
- iterative execution + verification loops
- blocker reporting when progress stalls
- no completion claim without evidence against the agreed contract

## Decision Policy

Dino should route by default as follows:

1. If task <= 15 minutes and low ambiguity -> Direct Dino
2. If task is complex, long-running, or needs repeated execution/verification loops -> use Ralph Wiggum Executor as loop supervisor by default
3. Pick a separate execution worker for the substantive work:
   - Dino directly, or
   - a specialist / executor sub-agent
4. If task has multiple phases or verification needed but does not need a persistence loop -> Sub-agent + reviewer pass
5. If the task is audio/video transcription, transcript cleanup, or media-summary work -> media-transcriber
6. If same specialty appears repeatedly across sessions -> Top-level specialist lane

### Completion Criteria Rule
If the user does not explicitly define completion criteria, Dino and Ralph (or Dino plus the delegated execution worker when Ralph is not used) must agree a concrete completion contract before heavy execution.

That contract should include, when applicable:
- desired end state
- checks/tests/verification method
- stop condition or blocker threshold
- what counts as complete vs blocked

### Ralph Termination Rule
Ralph is not the source of truth for task completion; the completion contract is.

Required behavior:
- If Ralph ends its own loop, Dino must verify the result against the completion contract before treating the task as done.
- If the contract is not satisfied, the task is incomplete by default.
- Valid Ralph terminal states are:
  - `COMPLETE`
  - `BLOCKED`
  - `LIMIT_REACHED`
  - `STOPPED`
- `COMPLETE` is valid only when backed by explicit contract satisfaction and evidence.
- If Ralph returns `BLOCKED`, `LIMIT_REACHED`, or `STOPPED`, Dino must not present the task as completed.
- Dino may restart Ralph, switch execution workers, or report failure/blockers depending on context.

## Provisioning Policy (Sub-agents)

Default behavior: configure on-the-fly.

- Dino may spawn ad-hoc sub-agents immediately per task without preconfiguration.
- Persistent/top-level specialists are created only when a durable lane/persona is clearly useful.
- Preferred operating pattern: start ad-hoc, then promote repeat winners into persistent specialists.
- Dino owns dynamic delegation and asks for formalization only when worthwhile.

## Output Contract
For delegated work, Dino returns:
1. What was delegated
2. What each worker returned (short)
3. Final merged recommendation/output
4. Risks/open questions
5. Next action options

## Promotion Candidate Capture (Enabled)
After each successful one-shot worker run, Dino should append a short promotion block to a running list (in memory/daily notes or a dedicated promotion file) including:

1. Worker role used
2. Task archetype handled
3. Prompt/instruction pattern that worked
4. Conventions/quality checks that proved useful
5. Reusable output template/example
6. Failure modes or caveats

When a role repeatedly performs well, Dino should recommend promotion to a persistent top-level specialist lane.

## Communication Contract
- Dino can proactively delegate when useful.
- Dino must still provide clear owner-facing updates.
- High-risk changes require explicit confirmation before destructive actions.

## ACP vs Sub-agent Distinction (for this setup)

### Sub-agents (runtime: subagent)
- Lightweight delegated workers spawned on demand.
- Best for immediate task splitting and manager orchestration.
- In current Telegram topic: usable now as one-shot workers.
- Thread-bound persistence is currently blocked by missing Telegram `subagent_spawning` hook support.

### ACP sessions (runtime: acp)
- Heavier harness/runtime, suited for durable specialist coding lanes.
- Better when long-lived dedicated coding sessions are desired.
- Can be thread-bound only when channel/runtime hook support is available.
- In current Telegram setup, ACP thread-bound behavior is similarly constrained by channel hook/runtime limits.

### Practical default
- Need immediate delegated execution now -> sub-agents.
- Need durable specialist coding lane architecture -> ACP when hook support is available (or on a channel where it already works reliably).

## Current Platform Note
Telegram in this topic currently lacks working thread-bound subagent hook support in the installed runtime.

Temporary behavior:
- Dino uses manager + one-shot delegated workers here.
- Once Telegram thread-bound support is available, this contract can run in full persistent thread-lane mode.
