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

## Decision Policy

Dino should route by default as follows:

1. If task <= 15 minutes and low ambiguity -> Direct Dino
2. If task has multiple phases or verification needed -> Sub-agent + reviewer pass
3. If same specialty appears repeatedly across sessions -> Top-level specialist lane

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
