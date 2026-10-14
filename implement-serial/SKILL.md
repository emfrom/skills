---
name: implement-serial
description: "Implement the result of /to-spec and /to-tickets in code, running only one agent at a time."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

The goal is the entire spec implemented on a single **integration branch**, with every ticket resolved the way the issue tracker closes work.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. Work through the tickets one at a time, respecting their blocking relationships.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Run only one agent at a time.** Never start another agent while one is still working. Wait for the current agent to finish before starting the next one.

## Steps

1. Read the spec and tickets to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future agents. Complete exploration before starting any other agent. This lets **implementer subagents** focus on implementation rather than exploration.

3. Create the integration branch. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first merge in step 5 (a branch with no commits ahead of main can't open one), marked as closing the spec and tickets.

4. Implement tickets serially using **one implementer subagent at a time**, each in its own worktree on its own branch. Choose a ticket whose blockers are resolved, and do not start another implementer until the current one has finished and been merged. Each implementer subagent:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - calls the Skill tool with `tdd` to build the ticket;
   - merges the integration branch tip into its own branch before reporting done

5. Once the current **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**. Wait for the merger to finish before starting another agent.

6. After each merge, update the ticket frontier and choose the next ready ticket. Start only one **implementer subagent** at a time, and wait for its completion and merge before proceeding to another ticket.

7. Once all tickets are complete, call the Skill tool with `code-review` on the integration branch. Fix all issues raised by the code review in a single **implementer subagent**. Run the review and fixer sequentially, never concurrently.

8. If a draft PR exists, mark it ready for review. Otherwise, resolve each ticket the way the issue tracker closes work, and report the integration branch.

9. Clean up all **implementer subagent** worktrees.
