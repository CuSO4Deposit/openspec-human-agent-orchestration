---
name: openspec-orchestration-uninstall
description: Remove the OpenSpec human-agent orchestration layer installed by `openspec-orchestration-init`. Use when a user wants to undo the project-local patches, remove sentinel blocks from OpenSpec propose/apply commands and agent guidance, delete or preserve generated execution-plan templates, or return to the default OpenSpec `propose -> apply` behavior without human-agent execution blocks.
---

# OpenSpec Orchestration Uninstall

## Purpose

Undo the project-local side effects created by `openspec-orchestration-init` while preserving user work by default.

This skill removes orchestration instructions from command prompts and agent guidance. It must not delete OpenSpec proposals, specs, tasks, source code, or completed change history unless the user explicitly asks.

## What To Remove

Remove sentinel blocks marked with:

```md
<!-- BEGIN openspec-human-agent-orchestration -->
...
<!-- END openspec-human-agent-orchestration -->
```

Check these files and roots when present:

- `.claude/commands/opsx/propose.md`
- `.claude/commands/openspec/propose.md`
- `.claude/commands/opsx/apply.md`
- `.claude/commands/openspec/apply.md`
- command files with equivalent OpenSpec propose/apply behavior
- `AGENTS.md` or the repository's equivalent agent guidance file
- `.codex/skills`
- `.agents/skills`
- `.claude/skills`
- `.cursor/skills`
- `skills`
- `agent-skills`
- any project-local skill directory the user names

Within skill roots, remove sentinel blocks only from OpenSpec-related skill files, including names that contain `openspec`, `opsx`, `open-spec`, `spec-change`, `proposal`, `apply`, or `propose`.
## Templates And Generated Plans

Handle generated files conservatively:

- Remove `openspec/templates/execution-plan.md` if it exactly matches the installed template or the user asks to remove templates.
- Do not delete `openspec/changes/*/execution-plan.md` by default. These may be useful project history.
- Offer to archive generated execution plans instead of deleting them if the user wants a cleaner tree.

## Workflow

1. Inspect the project for OpenSpec command files and guidance files.
2. Remove only the sentinel-marked orchestration blocks.
3. Check for `openspec/templates/execution-plan.md` and decide whether it is safe to remove.
4. Leave existing change-level `execution-plan.md` files in place unless explicitly told otherwise.
5. Report every command file, guidance file, and skill root searched.
6. Report every changed file and every skipped file.
7. Explain that the project now returns to the default OpenSpec workflow.

## Safety Rules

- Do not remove unmarked user text.
- Do not delete OpenSpec changes, specs, proposals, tasks, or source files.
- Do not run `rm -rf`.
- Prefer editing files in place with a clear summary.
- If a file contains a partial or malformed sentinel block, stop and ask before editing it.

## Final Response

Summarize:

- command files cleaned
- guidance files cleaned
- templates removed or preserved
- generated change plans preserved
- any manual cleanup needed
