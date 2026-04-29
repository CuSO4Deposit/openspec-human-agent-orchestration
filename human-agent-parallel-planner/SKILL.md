---
name: human-agent-parallel-planner
description: Create or repair the human-readable `execution-plan.md` used by OpenSpec human-agent orchestration. Normally invoked internally by a patched `opsx:propose` post-propose pass, not as an extra user workflow step. Also use directly when a user asks to regenerate an execution plan, add human gates, plan parallel code agents, define execution blocks, recover main-agent context, or create a bubble-squeezing plan without a machine-heavy YAML DAG.
---

# Human-Agent Parallel Planner

## Purpose

Turn an OpenSpec change or implementation task list into a human-readable execution plan that a main agent can reread before every execution block. In the normal installed workflow this skill is called from the patched propose command, so users still experience `propose -> human review -> apply` rather than `propose -> planner -> apply`. The plan should maximize safe parallel work between code agents and humans while keeping the normal human review step before implementation.

Do not implement code. Do not dispatch agents. Create or update the execution plan only.

## Inputs

Read the available source artifacts from disk, preferring this order:

1. OpenSpec change files: `openspec/changes/<change-id>/proposal.md`, `design.md`, `tasks.md`, and spec deltas.
2. Existing `execution-plan.md`, if present.
3. Project guidance such as `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, or command files.
4. Relevant source tree structure only when needed to assign ownership boundaries.

If the change id is unclear, infer it from the user's request or ask one concise question.

## Output File

Write the plan to:

`openspec/changes/<change-id>/execution-plan.md`

If the project is not using OpenSpec, write to the nearest task-specific planning directory and name it `execution-plan.md`.

## Planning Principles

- Prefer a natural-language, time-ordered collaboration plan over a long YAML DAG.
- Treat humans as first-class participants, not only final reviewers.
- Put human-readable orchestration first; avoid raw dependency tables unless explicitly requested.
- Express dependencies through block order and `Start when` conditions.
- Ask human questions as early as possible to reduce downstream idle time.
- Use read-only discovery blocks before write-heavy implementation blocks when ownership is uncertain.
- Assign disjoint write ownership to implementation agents. If ownership overlaps, serialize the work.
- Put public API, schema, auth, billing, permission, destructive, or broad refactor changes behind explicit stop conditions or human gates.
- Keep the plan concise enough for a main agent to reread every round.

## Required File Structure

Use this structure for `execution-plan.md`:

```md
# Execution Plan: <change-id>

Status: draft_pending_human_review

## How To Use This Plan

This plan is the source of truth for apply. The main agent must reread it before every block, summarize the current position, execute only the next unblocked block, and update progress after the block.

Human gates can only be completed by explicit human approval.

## Current Position

- Current block: Block 0 — Human review and approval
- Completed blocks: none
- Active work: none
- Waiting on human: approval of this execution plan
- Next unblock: approval starts Block 1

## Block 0: Human Review and Approval

Start when:
- OpenSpec proposal, design, tasks, and this execution plan have been generated.

Human does:
- Review the OpenSpec proposal, design, tasks, and this execution plan.
- Confirm whether agent split, human gates, and risk stops are acceptable.

Main Agent does:
- Does not implement code.
- Answers questions and revises the plan if requested.
- Waits for explicit approval.

Output:
- Approved execution plan.

Stops until:
- Human explicitly approves.
```

Then add as many blocks as needed, usually:

- Block 1: Parallel discovery
- Block 2: Human decisions before implementation, if needed
- Block 3: Parallel implementation
- Block 4: Integration
- Block 5: Verification and final review

## Block Format

Each block must include:

```md
## Block N: <name>

Start when:
- <conditions that must be true>

Can run in parallel because:
- <why participants do not conflict>

<Participant Name>:
- Goal: <what this participant accomplishes>
- Scope: <read/write ownership or human decision scope>
- Produces: <artifact, summary, patch, decision, or approval>
- Stops if: <conditions that require human or main-agent intervention>

Human:
- <what the human does during this block, or "Not needed unless a stop condition triggers.">

Output:
- <block-level output>

Progress:
- Not started.
```

Omit `Can run in parallel because` for serial blocks.

## Participant Types

Use clear participant names:

- `Human`
- `Main Agent`
- `Backend Explorer Agent`
- `Frontend Explorer Agent`
- `Test Explorer Agent`
- `Backend Implementation Agent`
- `Frontend Implementation Agent`
- `Verification Agent`
- Domain-specific names when more accurate

For every code agent, state:

- goal
- read scope
- write scope, if any
- forbidden files/modules, if important
- output expected by the main agent
- stop conditions

## Human Gates

Human gates must be plain-language blocks or participant entries. Each gate must say:

- the question or approval needed
- why it matters
- what work it blocks
- acceptable options, if known
- whether a default exists if the human defers

Never mark a human gate complete unless the user explicitly approved or answered it in the conversation.

## Current Position Updates

When updating an existing plan, update only:

1. `Status`
2. `Current Position`
3. the `Progress` entries for completed or active blocks
4. any newly discovered questions, gates, risks, or ownership changes

Keep historical progress short. The plan is working memory, not a full transcript.

## Approval States

Use simple status values:

- `draft_pending_human_review`
- `approved`
- `in_progress`
- `blocked_on_human`
- `complete`

`apply` must not start implementation while status is `draft_pending_human_review`.

## Final Response

After creating or updating the plan, summarize:

- where the plan was written
- the current block
- which agents would run in the first parallel block
- what human input or approval is needed now
- that implementation has not started
