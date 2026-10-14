---
name: automate-spec
description: "Run an end-to-end, low-interruption implementation of a configured repository spec and its ticket graph. Explicitly invoke when you want all independently closable tickets implemented and integrated, with unresolved work documented rather than blocking the rest."
disable-model-invocation: true
---

# Automate a spec

This skill defines **how to run** the work; `implement-spec` defines **what implementation process to run**. Load and follow the `implement-spec` skill as the canonical implementation procedure, including its integration-branch, worktree, implementer, merger, review, and ticket-resolution steps. Apply this skill's run policy to keep the orchestration moving across local blockers and to maximize completed tickets. Never invent facts, claim unverified work is complete, or expand scope.

## 1. Establish the run policy

Read the repository's agent instructions, Matt Pocock configuration, spec, tickets, and any linked research or decision records. Follow the configured issue tracker and ticket dependency graph.

Look for `docs/agents/automation-policy.md`. If it exists, use it as the run's source of truth. If it does not, create it from the following proposed defaults and show the user the created policy and its boundaries before starting implementation. Do not turn ordinary implementation uncertainty into a request for user input; use the resolution loop below.

Proposed initial policy:

- **Scope:** only changes required to implement the accepted spec and its tickets, plus tests and docs necessary to validate and explain those changes.
- **Filesystem boundary:** modify files inside the repository. Put temporary notes, logs, and scratch artifacts in `/tmp`. Use temporary Git worktrees under `/tmp` (or inside the repository's designated worktree directory, if configured). Do not modify other paths.
- **Preservation:** preserve pre-existing user changes. Keep each implementation isolated in its own worktree/branch; never overwrite unrelated work.
- **External effects:** use only the issue tracker and repository hosting configured for this repository, and only the actions required by its documented ticket/PR workflow. Do not publish, deploy, release, or alter credentials.
- **Execution:** complete every ticket whose dependencies and evidence permit completion. Continue through independent tickets when another ticket is blocked. Close/resolve only tickets whose acceptance criteria are verified.
- **Parallelism and models:** use configured implementer/oracle models and concurrency limits. If absent, use the available default implementer model, a modest safe concurrency, and an available independent oracle for difficult decisions; don't assume model names or resource-priority tools exist.
- **Stopping:** do not stop the overall run for a local blocker. Keep a ticket open when it cannot be completed truthfully or safely; record the exact blocker and continue the remaining frontier. Stop only when there is no useful in-scope work left, or a hard environment/policy boundary prevents all further progress.

The policy is a durable repository configuration. Keep it concise, editable, and specific to the repo. If a necessary policy value is unspecified, apply the conservative defaults above and record the assumption; do not suspend implementation over minor choices.

## 2. Build the ticket frontier

Read the full spec and all associated tickets. Understand acceptance criteria and dependencies before dispatching work. If the issue tracker lacks a complete ticket graph, use the repo's configured ticket workflow to identify the actionable work; do not invent extra tickets or broaden the spec.

Create or reuse the configured integration branch without disturbing existing work. Identify all tickets whose blockers are resolved. Dispatch independent ready tickets concurrently, respecting the run policy and available capacity. Each implementer works in an isolated worktree based on the current integration branch and invokes the `tdd` skill to implement its ticket test-first, as required by `implement-spec`. Each reports tests, changes, and unresolved concerns.

Use context pointers (ticket/spec/research references) in agent handoffs. Keep handoffs bounded and avoid repeating source material already available to the implementer.

## 3. Resolve problems without stalling

For each error, ambiguity, conflict, or failed validation:

1. Capture the exact symptom and inspect relevant repo guidance, code, history, tests, and tool output.
2. Try the smallest in-scope repair or evidence-gathering step. Use a bounded oracle consultation for difficult diagnosis or design choices; seek a second oracle only if the first cannot resolve it.
3. Choose the most conservative option supported by repo evidence and the accepted spec. Preserve existing behavior where intent is unclear. Do not invent user facts, silently weaken acceptance criteria, or introduce unrelated work.
4. Retry validation and continue all independent tickets.
5. If the ticket remains blocked, leave it unclosed and write a concise blocker record with what was tried, evidence, and the smallest next action needed. Then proceed with the rest of the frontier.

A blocker is local to its ticket unless evidence shows it invalidates other tickets. Re-evaluate the frontier after each merge or blocker update. Never report partial implementation as full completion.

## 4. Integrate and validate continuously

When an implementer finishes, inspect its report and changes. Verify its worktree is based on the latest integration branch, merge the integration tip into the work branch if needed, then review and merge the completed work into the integration branch. Resolve merge conflicts conservatively using the spec, repository conventions, and oracle support; rerun relevant tests after every merge.

After each merge, start newly unblocked tickets. Avoid parallel work on overlapping files unless the integration strategy is clear. If an implementer fails or becomes unavailable, recover its work and reassign only the remaining work; don't restart completed tickets without cause.

## 5. Close out

When no ready tickets remain, perform the configured full validation and code review on the integration branch. Fix in-scope findings and rerun validation. Resolve only tickets whose acceptance criteria are verified, using the configured issue-tracker workflow. Leave genuinely blocked tickets open with precise evidence-based notes.

Clean up only worktrees created by this run, and only after preserving useful logs/results in `/tmp` or the repo. Report the integration branch, completed and resolved tickets, validation, and any remaining blocked tickets. State explicitly that the spec is fully complete only if every ticket and acceptance criterion is verified.
