---
name: openspec-orchestration-init
description: Upgrade an OpenSpec-initialized project with human-agent orchestration. Use after `openspec init` or when a project has OpenSpec/opsx commands and the user wants to keep the normal `propose -> human review -> apply` workflow while adding post-propose execution plans, parallel agent blocks, human gates, persistent main-agent context, and DAG-aware apply behavior. This skill patches generated OpenSpec command prompts and project guidance; it does not apply a specific change.
---

# OpenSpec Orchestration Init

## Purpose

Install an orchestration layer on top of OpenSpec so users can keep this workflow:

`opsx:propose -> human review -> opsx:apply`

After the upgrade, users still run `propose -> human review -> apply`. `propose` still generates the normal OpenSpec artifacts, then performs a post-propose orchestration pass that writes a human-readable `execution-plan.md`. `apply` refuses to implement until that plan is approved, then rereads and updates it before every execution block.

## What To Patch

Patch only OpenSpec commands and OpenSpec skill files inside the current repository. Do not suggest or perform patching of skills or files outside the current repository.

Always inspect these project-local command files when present:

- `.claude/commands/opsx/propose.md`
- `.claude/commands/openspec/propose.md`
- `.claude/commands/opsx/apply.md`
- `.claude/commands/openspec/apply.md`
- command files with equivalent OpenSpec propose/apply behavior
- `AGENTS.md` or the repository's equivalent agent guidance file

Also inspect these skill roots when they exist, because OpenSpec may be installed as skills rather than command files:

- `.codex/skills`
- `.agents/skills`
- `.claude/skills`
- `.cursor/skills`
- `skills`
- `agent-skills`
- any project-local skill directory the user names
Within each skill root, look for OpenSpec-related skill folders or files, including names that contain `openspec`, `opsx`, `open-spec`, `spec-change`, `proposal`, `apply`, or `propose`. Patch the OpenSpec propose/apply instructions wherever they actually live.

If command files are absent but OpenSpec skill files are found, patch the skill files and report that command-specific patching was skipped. If neither command files nor OpenSpec skill files are found, add fallback guidance to `AGENTS.md` and report what was searched.


## OpenSpec Skill Discovery And Patch Search

Do not assume OpenSpec behavior lives only in `.claude/commands`. In many setups it lives in project-local skill files under `.codex/skills`, `.agents/skills`, or a vendored skills tree.

Before declaring the project already initialized, search the command paths and skill roots listed above. For each OpenSpec-related file found, classify it as one of:

- propose instructions
- apply instructions
- combined OpenSpec workflow instructions
- unrelated reference material

Patch propose/apply instructions with the sentinel block. Do not patch unrelated reference material unless it is clearly used as the active command/skill instruction.

When initializing or re-running initialization, check and report:

- command patches present and current
- OpenSpec skill files searched
- OpenSpec skill files patched or already current
- agent guidance present and current
- `openspec/templates/execution-plan.md` present
- whether the propose patch contains a self-contained fallback instead of relying only on helper skill invocation

## Patch Strategy

Use additive, idempotent sentinel blocks. Do not replace the original OpenSpec workflow.

Use these markers:

```md
<!-- BEGIN openspec-human-agent-orchestration -->
...
<!-- END openspec-human-agent-orchestration -->
```

If a marker already exists, inspect its contents and update the block in place when this skill's current protocol is stronger than the installed block. Do not report "already initialized" until command patches, agent guidance, template installation, and self-contained planner fallback have all been checked. If a marker does not exist, append the block to the relevant section or file.

## Propose Patch

Add this behavior to the end of the propose command:

```md
<!-- BEGIN openspec-human-agent-orchestration -->

## Post-Propose Human-Agent Execution Plan

After the normal OpenSpec proposal, design, specs, and tasks are written:

1. Re-read the generated OpenSpec files from disk. Do not rely only on chat context.
2. Create or update `openspec/changes/<change-id>/execution-plan.md`.
3. Use the `human-agent-parallel-planner` skill if it is available in the current runtime. If it is not available or not discovered, continue with the inline instructions in this block; do not fail and do not ask the user to install another skill during propose.
4. Use a human-readable, block-by-block plan, not a long YAML DAG.
5. Include:
   - `Status: draft_pending_human_review`
   - `How To Use This Plan`
   - `Current Position`
   - time-ordered execution blocks
   - what each code agent and the human do in each block
   - start conditions for every block
   - why participants in a block can run in parallel
   - expected outputs for every participant
   - human long-running work, human gates, and human input tasks
   - stop conditions for risky work
   - agent ownership boundaries
6. Required block structure:
   - `Block 0: Human Review and Approval`
   - discovery blocks before implementation when ownership is uncertain
   - human decision blocks before blocked implementation work
   - parallel implementation blocks only when write scopes do not overlap
   - integration blocks after parallel work
   - verification and final human review blocks
7. Each block must include `Start when`, participant goals/scopes/outputs, stop conditions, and a short `Progress` entry.
8. The first block must be human review and approval.
9. Do not start implementation during propose.
10. End by telling the user to review the OpenSpec files and `execution-plan.md` before apply.

<!-- END openspec-human-agent-orchestration -->
```

The propose patch must not rely on `human-agent-parallel-planner` being discoverable. It should use that skill internally when available, but must include enough inline instructions to produce `execution-plan.md` without it. Users should not need to run the planner as a separate step in the normal workflow.

## Apply Patch

Add this behavior to the beginning of the apply command:

```md
<!-- BEGIN openspec-human-agent-orchestration -->

## Human-Agent Execution Plan Requirement

Before implementing any OpenSpec task:

1. Read `openspec/changes/<change-id>/execution-plan.md` from disk.
2. Summarize the current position in every response before taking action:
   - status
   - current block
   - completed blocks
   - active work
   - waiting on human
   - next unblock
3. If the plan is missing, stop and ask the user to run propose again or generate the execution plan.
4. If `Status` is `draft_pending_human_review`, stop and ask for explicit approval. Do not implement.
5. If `Status` is `approved` or `in_progress`, execute only the next unblocked block.
6. If `Status` is `blocked_on_human`, stop and ask for the required decision or approval.
7. Never mark human gates complete unless the human explicitly approved or answered.
8. Reread `execution-plan.md` before every block and update `Current Position` plus block `Progress` after every block.
9. When dispatching subagents, give each one its block name, goal, start conditions, scope, expected output, and stop conditions from the plan.
10. Do not implement directly from `tasks.md` unless the user explicitly overrides this protocol.

<!-- END openspec-human-agent-orchestration -->
```

## Agent Guidance Patch

Add this to `AGENTS.md` or the project guidance file:

```md
<!-- BEGIN openspec-human-agent-orchestration -->

## OpenSpec Human-Agent Orchestration

When working with OpenSpec changes, preserve the normal human review workflow:

`propose -> human review -> apply`

After propose, generate `openspec/changes/<change-id>/execution-plan.md`. This plan must be human-readable and organized by time-ordered blocks. Each block describes start conditions, participating agents, human gates, goals, scopes, outputs, and stop conditions.

During apply, the main agent must use `execution-plan.md` as external working memory. Before every block, reread the file and state the current position. After every block, update `Current Position` and the block's `Progress` entry.

Do not start implementation if the plan status is `draft_pending_human_review`. Human gates can only be completed by explicit human approval.

<!-- END openspec-human-agent-orchestration -->
```

## Execution Plan Template

Install or reference the template at `openspec/templates/execution-plan.md` when useful. Use the template in `assets/execution-plan-template.md` from this skill if available.


## Related Skills

- `human-agent-parallel-planner` is the internal planning workflow used by the patched propose command to create `execution-plan.md`. It is also useful for advanced users who want to regenerate or repair an execution plan manually.
- `openspec-orchestration-uninstall` removes the sentinel blocks, templates, and guidance added by this skill.

## Validation

After patching:

1. Show the files changed.
2. Confirm whether propose command patching succeeded.
3. Confirm whether apply command patching succeeded.
4. List project-local skill roots searched, including `.codex/skills` when present.
5. List OpenSpec skill files patched or already current.
6. Confirm whether agent guidance patching succeeded.
7. Confirm whether `openspec/templates/execution-plan.md` exists.
8. Confirm whether the propose patch is self-contained and does not depend on helper skill discovery.
9. Explain the resulting user workflow in one short example.

Do not run OpenSpec commands unless the user asks.
