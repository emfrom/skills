---
name: automate-spec
description: "Run a low-interruption implementation of a configured repository spec and its ticket graph, integrating independently closable tickets and documenting blockers."
disable-model-invocation: true
---

# Automate a spec

This skill defines run orchestration; `implement-spec` defines the canonical implementation procedure, including worktrees, implementers, integration, review, and ticket resolution. Load and follow it. This skill supplies bounded permissions, progress tracking, and recovery rules. Never claim unverified work complete or expand the accepted scope.

## 1. Establish policy and permissions

Read repository agent instructions, Matt Pocock configuration, the spec, all associated tickets, and linked research/decision records. Follow the configured issue tracker and ticket dependency graph.

Read `docs/agents/automation-policy.md` if present. Treat it as the repository's run policy, subject to the permission boundaries below. If absent, use the defaults here as **run-local policy**; do not create repository policy files just to start. If a repository policy is needed for future runs, propose it separately at closeout.

**Authorization is action-specific.** A request to automate, continue, or avoid interruptions authorizes in-scope repository implementation and its documented local validation only. It does not by itself authorize authentication, use of credentials, live service access, network requests with external effects, publishing, deployment, release, pushing, destructive cleanup, or changes outside the repository. Perform such an action only when the current user request or existing repository policy explicitly authorizes that specific class of action. Keep credentials private. Never interpret an authorization as unlimited retries: apply the retry and resource limits below. When approval is required, pause only that action, record the exact approval needed, and continue independent work.

Defaults:

- **Scope:** accepted spec/tickets, and necessary tests and documentation.
- **Filesystem:** modify repository files and run-created worktrees only. Put run ledger, logs, and scratch artifacts in a unique directory under `/tmp`. Preserve pre-existing user changes; inspect status before branch/worktree operations.
- **External effects:** only explicitly authorized actions, using the repository's documented workflow. Never alter credentials or publish/deploy/release without specific authorization.
- **Progress:** implement every actionable ticket whose dependencies and evidence permit completion; continue independent work past local blockers. A blocked ticket remains open with evidence.
- **Parallelism and roles:** use a configured `worker` for ticket implementation and a subagent named `delegate` for non-implementation tasks such as research, diagnosis, or independent review. Use an oracle for consequential design/diagnosis decisions when available. Obey configured agent/model and concurrency limits; otherwise use at most two concurrent implementation tickets. If the needed role is unavailable, proceed directly when appropriate or record the blocker—never assume a generic default agent or invent model names.
- **Retries/resources:** for a retryable operation, make at most three attempts with bounded backoff, unless repository policy specifies a stricter limit. Do not retry authorization, credential, or permanent failures automatically. Keep at most two implementation workers active; avoid unbounded jobs or output. If a resource limit or environment constraint prevents progress, record it and continue elsewhere.
- **Stopping:** stop when no useful in-scope work remains or a hard boundary blocks all progress. Do not stop the whole run for a ticket-local blocker.

## 2. Create a resumable run ledger

Create `/tmp/automate-spec-<repo>-<run-id>/run.md` (choose a unique, filesystem-safe repository name and run ID). Keep it concise and update it after dispatch, completion/blocking, integration, and any interruption. It is the run's recovery record, not a substitute for repository tickets.

Record:

- repository path, starting branch/commit, integration branch, policy source, and authorized external-action classes;
- each ticket's status (`ready`, `active`, `integrated`, `blocked`, or `resolved`), dependencies, worker/worktree/branch, latest verified commit, acceptance evidence, validation commands/results, blocker, and next action;
- run-level validation and review results, plus worktrees created by this run.

Before resuming after interruption, read the ledger, inspect actual Git/worktree and ticket state, and reconcile discrepancies from current evidence. Never assume a worker or command completed merely because the ledger says it was started. Completion criterion: the ledger agrees with the repository's observable state and names the next actionable work.

## 3. Build the ticket frontier

Read the full spec and all associated tickets. Capture each ticket's acceptance criteria and dependencies in the ledger. A ticket is independently closable when its own criteria can be implemented and verified without depending on unresolved work; shared foundations may still impose dependencies. Follow explicit dependencies. If dependencies are missing, infer only clear relationships from acceptance criteria and record the inference. If the graph is cyclic or materially ambiguous, identify the smallest prerequisite slice supported by evidence; otherwise mark affected tickets blocked and continue elsewhere. Do not invent scope or tickets.

Inspect Git status and existing branches/worktrees before creating or reusing an integration branch. Preserve existing work. Reuse a branch only when its purpose and state are clear; otherwise use the integration procedure in `implement-spec` and record the choice. Dispatch independent ready tickets concurrently within the limit, using a `worker` for each implementation ticket. Give each worker a bounded ticket, acceptance criteria, relevant context pointers, and an isolated worktree based on the current integration branch. Use a subagent named `delegate` for bounded non-implementation work when delegation is authorized and useful; do not assign implementation to a delegate subagent as a substitute for a worker. Require test-first implementation as specified by `implement-spec` and a report of changes, exact checks run/results, commit, and unresolved concerns.

## 4. Resolve problems without stalling

For each failure, ambiguity, conflict, or failed validation:

1. Record the exact symptom and inspect relevant instructions, code, history, tests, and output.
2. Try a small in-scope repair or evidence-gathering step. For difficult design/diagnosis, consult one independent oracle; seek another only if the first cannot resolve it within the retry/resource limits. Use a `delegate subagent` for bounded research or review tasks, not implementation.
3. Treat oracle output as advice, not evidence or authorization. Verify claims against repository/source/test evidence. The agent remains responsible for policy and permission checks.
4. Choose the most conservative option supported by the accepted spec and repository evidence. Preserve existing behavior when intent is unclear; never weaken acceptance criteria to make a check pass.
5. Retry validation within the stated limits and continue independent tickets.
6. If unresolved, leave the ticket open and record evidence, attempts, and the smallest next action. Continue the frontier.

## 5. Integrate and validate continuously

When an implementer finishes, inspect its report, Git diff, and worktree state. Confirm the work is based on the latest integration branch; follow `implement-spec` to update, review, and merge it. Resolve conflicts conservatively using the spec and repository conventions; use oracle advice where helpful, but verify the result. Run relevant checks after each merge and record exact commands and outcomes. Then update the ledger and dispatch newly unblocked tickets. Avoid parallel work on overlapping files unless the integration plan is explicit. If a worker fails or becomes unavailable, inspect and preserve its work; reassign only unfinished work.

Use the repository's documented validation and review workflow. If none exists, run the package scripts or documented project checks discoverable from the repository, then perform a focused diff review and an independent review when available. Record unavailable or failing checks accurately; do not claim validation passed. Do not change tests or policy merely to conceal a failure.

## 6. Close out

When no ready tickets remain, perform the configured full validation and code review on the integration branch. Fix in-scope findings, rerun affected validation, and update the ledger. Resolve tickets only when every acceptance criterion has recorded verification evidence, following the configured tracker workflow and the authorization boundary. A local commit is not the same as a push or tracker resolution; perform those only when specifically authorized.

Clean up only worktrees created by this run, after confirming their useful changes/results are preserved and following the filesystem and authorization boundaries. Keep the ledger and useful logs in its `/tmp` run directory. Report the integration branch and commit, tickets integrated/resolved/blocked, exact validation and review results, remaining risks, and any action awaiting approval. State that the spec is complete only when every ticket and acceptance criterion is verified and the configured resolution workflow is complete.
