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

Progress:
- Not started.

## Block 1: Parallel Discovery

Start when:
- Block 0 is approved.

Can run in parallel because:
- Discovery agents are read-only.
- Each agent inspects a separate concern.

<Domain> Explorer Agent:
- Goal: inspect <domain> boundaries, relevant files, and risks.
- Scope: read-only <paths or areas>.
- Produces: findings, recommended ownership, risks, and questions.
- Stops if: it finds ambiguity that blocks implementation.

Human:
- Can answer product or technical questions asynchronously while discovery runs.
- Does not block discovery unless a question is required before discovery.

Output:
- Discovery summaries.
- Confirmed implementation ownership.
- Updated risk list.

Progress:
- Not started.

## Block 2: Human Decisions Before Implementation

Start when:
- Discovery is complete.
- Required questions are known.

Human Gate:
- Question: <question>
- Needed because: <why this changes implementation>
- Blocks: <work blocked by this answer>
- Acceptable answers: <options, if known>
- Default if deferred: <default or "no default; implementation waits">

Main Agent:
- Summarizes discovery findings.
- Recommends a decision when appropriate.
- Waits for human answer.

Output:
- Human decisions recorded.
- Implementation can start.

Progress:
- Not started.

## Block 3: Parallel Implementation

Start when:
- Discovery is complete.
- Required human decisions are answered.
- Ownership boundaries do not overlap.

Can run in parallel because:
- Implementation agents have disjoint write scopes.
- Shared contracts were agreed before this block.

<Domain> Implementation Agent:
- Goal: implement <domain behavior>.
- Owns: <paths>.
- Must not edit: <paths>.
- Produces: patch and integration notes.
- Stops before: schema migration, public API change, auth/permission change, billing change, destructive operation, or scope expansion.

Human:
- Not needed unless a stop condition triggers.

Output:
- Implementation patches.
- Integration notes.

Progress:
- Not started.

## Block 4: Integration

Start when:
- Parallel implementation outputs are available.

Main Agent:
- Integrates parallel outputs.
- Resolves conflicts.
- Ensures behavior matches OpenSpec.
- Updates OpenSpec tasks and this plan.
- Does not expand scope.

Human:
- Needed only if integration reveals a product tradeoff or risk gate.

Output:
- Integrated implementation.

Progress:
- Not started.

## Block 5: Verification and Final Review

Start when:
- Integration is complete.

Verification Agent:
- Runs targeted tests and checks.
- Reports failures, regressions, and confidence.
- Does not perform broad unrelated refactors.

Main Agent:
- Fixes integration issues only.
- Updates task checkboxes and this plan.
- Summarizes final result.

Human Gate:
- Human reviews final behavior.
- Human approves completion or requests changes.

Output:
- Completed change or follow-up fix block.

Progress:
- Not started.
