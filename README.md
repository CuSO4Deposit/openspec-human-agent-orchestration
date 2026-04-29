# OpenSpec Human-Agent Orchestration Skills

OpenSpec is great at turning a proposed change into specs and tasks. The default workflow, however, usually treats `apply` as a mostly linear implementation step: once the human has reviewed the proposal, the agent starts working through tasks.

This skill pack adds a lightweight orchestration layer for teams that want to keep the familiar OpenSpec flow while making human/agent collaboration more explicit:

```text
opsx:propose -> human review -> opsx:apply
```

After installation, `propose` still creates the normal OpenSpec artifacts, then adds a human-readable `execution-plan.md` that explains which agents and humans participate in each time block. `apply` then uses that plan as persistent working memory, rereading it before every block and stopping at human gates.

## The Pain

Real implementation work is not just agents waiting for a human to say yes. Humans often have their own long-running work in the loop: reviewing product tradeoffs, checking designs, validating domain assumptions, testing a staging build, getting stakeholder feedback, preparing data, or making API/schema decisions.

OpenSpec task lists usually do not encode that collaboration shape:

- which agent tasks can safely run while the human is doing long-running review or research
- which human tasks are true blockers versus work that can happen in parallel
- when the main agent should ask early questions to avoid downstream idle time
- which tasks can safely run in parallel across multiple code agents
- which agent owns which files or modules
- where implementation should stop for risky changes
- how the main agent should recover context between rounds
- what each parallel work block is trying to produce

Without that structure, `apply` can become either too linear or too eager. It may wait unnecessarily for human work that could have happened alongside read-only discovery, miss chances to parallelize agent work while the human investigates something, or continue past places where a human's in-progress work should become the next input.

This project addresses that gap by making the post-propose review artifact explicit and readable by both humans and agents: a shared collaboration plan, not just an approval checklist.

## Installation

Recommended: copy this prompt to your code agent and let it install the skills for you:

```text
Install the OpenSpec Human-Agent Orchestration skills from https://github.com/CuSO4Deposit/openspec-human-agent-orchestration into my skills directory.

Please:
1. Clone or download the repository.
2. Copy these skill directories into the active skills directory for my code agent:
   - `openspec-orchestration-init`
   - `human-agent-parallel-planner`
   - `openspec-orchestration-uninstall`
3. Do not modify my current project yet.
4. Tell me to restart or start a new agent session so the skills are discovered.
```

<details>
<summary>Manual installation</summary>

Clone the repository and copy the skills into your agent's skills directory:

```bash
git clone https://github.com/CuSO4Deposit/openspec-human-agent-orchestration.git
cd openspec-human-agent-orchestration
mkdir -p "${AGENT_SKILLS_DIR:-$HOME/.codex/skills}"
cp -R openspec-orchestration-init \
      human-agent-parallel-planner \
      openspec-orchestration-uninstall \
      "${AGENT_SKILLS_DIR:-$HOME/.codex/skills}/"
```

Restart or start a new agent session so the skills are discovered.

If your runtime uses a different skills path, copy the three skill directories into that runtime's skill discovery directory.

</details>

## What Gets Installed

This repository contains three portable code-agent skills:

### `openspec-orchestration-init`

Use once after `openspec init` in a project.

It patches project-local OpenSpec/opsx command prompts and agent guidance so:

- `propose` generates a draft `execution-plan.md` after normal OpenSpec files are written
- `apply` refuses to implement until the execution plan is approved
- the main agent rereads the execution plan before every block
- human gates can only be completed by explicit human approval

### `human-agent-parallel-planner`

Normally invoked internally by the patched `propose` command.

It creates or repairs:

```text
openspec/changes/<change-id>/execution-plan.md
```

The plan is intentionally natural-language-first, not a large YAML DAG. It is organized as time-ordered blocks with start conditions, participants, goals, scopes, outputs, and stop conditions.

### `openspec-orchestration-uninstall`

Use to remove the orchestration layer.

It removes only the sentinel-marked blocks added by the init skill and preserves OpenSpec changes, specs, tasks, source code, and generated change-level execution plans by default.

## Project Setup

In each OpenSpec project:

```bash
openspec init
```

Then ask your code agent:

```text
Use openspec-orchestration-init to upgrade this project.
```

The skill patches project-local files only.

## Daily Workflow

After setup, keep using the normal OpenSpec workflow:

```text
opsx:propose <change>
```

The patched propose flow generates the usual OpenSpec artifacts plus:

```text
openspec/changes/<change-id>/execution-plan.md
```

Review the proposal, design, tasks, and execution plan. The execution plan explains:

- what happens in each block
- which agents can run in parallel
- what humans do in each block
- what input or approval is needed
- what risky work will stop for review

After approving the plan, continue as usual:

```text
opsx:apply <change>
```

During apply, the main agent must summarize its current position before each block, for example:

```text
Current position:
- Status: in_progress
- Current block: Block 3 — Parallel Implementation
- Completed blocks: Block 0, Block 1, Block 2
- Active work: Backend Implementation Agent, Frontend Implementation Agent
- Waiting on human: none
- Next unblock: both implementation agents complete
```

## Execution Plan Shape

The generated plan is designed to be read by humans and agents:

```md
# Execution Plan: <change-id>

Status: draft_pending_human_review

## Current Position

- Current block: Block 0 — Human review and approval
- Completed blocks: none
- Active work: none
- Waiting on human: approval of this execution plan
- Next unblock: approval starts Block 1

## Block 1: Parallel Discovery

Start when:
- Block 0 is approved.

Can run in parallel because:
- Discovery agents are read-only.
- Each agent inspects a separate concern.

Backend Explorer Agent:
- Goal: inspect backend boundaries, routes, services, persistence, and risks.
- Scope: read-only backend files.
- Produces: findings, recommended ownership, risks, and questions.
- Stops if: schema, auth, billing, or public API ambiguity is found.

Human:
- Can answer product questions asynchronously while discovery runs.
```

The plan avoids a machine-heavy DAG by expressing dependencies through block order and `Start when` conditions.

## Uninstall

Ask your code agent:

```text
Use openspec-orchestration-uninstall to remove the OpenSpec human-agent orchestration layer from this project.
```

The uninstall skill removes sentinel-marked orchestration patches and preserves generated change-level `execution-plan.md` files unless you explicitly ask to delete or archive them.

## Design Principles

- Keep the user workflow as `propose -> human review -> apply`.
- Make the coordination plan readable by humans, not just machines.
- Use `execution-plan.md` as external memory for the main agent.
- Prefer time-ordered blocks over large YAML DAGs.
- Treat human gates as first-class blockers.
- Parallelize read-only discovery before write-heavy implementation.
- Assign disjoint write scopes to parallel implementation agents.
- Stop before risky changes rather than letting apply continue silently.

