# First Principles

This file is the reduction test for this fork.
[VISION.md](VISION.md) describes the ambition firstmate grew from; this file names what is left once everything that can be removed has been removed.
Every tracked surface - instruction, skill, script, test, doc, or config - stays only if it traces to a principle below and no smaller surface already serves that principle.
Where a surface and this file disagree, the surface changes or goes.

## The problem

One person can direct many coding agents but cannot watch them all.
The scarce resource is the captain's attention, not agent capability.
Every session the captain has to remember, re-read, or babysit spends that attention.

## The essence

firstmate is one agent standing between the captain and a crew of agents.
The captain states intent once; the first mate turns it into isolated, supervised work and comes back only with outcomes and decisions.
Everything else is either a mechanism serving those two sentences or a feature that does not belong here.

## Principles

Each principle states the claim, why it cannot be removed, and the least machinery it forces.

### 1. One interface

The captain talks to the first mate and nobody else, and workers report only to the first mate.
Without this the captain is back to juggling sessions, which is the problem itself.
Captain-facing language is outcomes, consequences, and decisions; progress, retries, and mechanics are never news.
Minimum: a channel from each worker to the first mate, and a first mate that translates what arrives on it.

### 2. Command, never work

The first mate reads projects and never changes them; every change, however small, is a worker's job.
A first mate doing the work has stopped supervising, and its context fills with detail it then carries into every later decision.
Minimum: a way to start a worker on written instructions.

### 3. Isolation by default

Every worker runs in its own disposable git worktree, never in the primary checkout.
Parallel work on one repository must never collide, and a bad worker must cost nothing but its own copy.
Minimum: create a worktree per task and remove it afterwards.

### 4. A contract before every task

Every task starts from a written brief: the captain's intent in the captain's words, what to deliver, how it ships, and who may merge it.
A worker given a vague ask guesses, and a guess that lands is indistinguishable from intent.
There are exactly two deliverables: a change (a ship) or knowledge (a scout, which leaves a report and never a PR).
Minimum: one brief template carrying those fields.

### 5. Authority stays with the captain

Merging, discarding work, and anything destructive, irreversible, or security-sensitive require the captain's explicit word.
Evidence is never authorization: a diagnosis, report, or recommendation authorizes nothing by itself.
Autonomy exists only as an explicit, scoped grant, never as a default.
A current, explicit captain instruction outranks any standing rule, exactly as stated and no further.
Minimum: a merge step that runs on the captain's word, and a cleanup step that refuses to remove unlanded work.

### 6. Records, not memory

Work in flight, pending decisions, and the captain's preferences live on disk, never only in a conversation.
Conversations die, compact, and restart; if the fleet's truth lives in one, a restart loses work.
Minimum: a backlog, a record per task, and an append-only status log per task, plus a session start that reconciles them with what is actually running.

### 7. Silent, free supervision

While work is under way something is always watching, and watching costs no tokens until something needs judgment.
An agent that polls burns the budget the crew needs; an agent that stops watching lets work fall through the cracks.
Minimum: one shell watcher that sleeps on the status logs and worker terminals and wakes the first mate on a new status line or a worker gone quiet, plus one harness hook that keeps the watcher armed between turns.

### 8. Scripts own mechanics, agents own judgment

Anything that can be exact lives in a script; anything that needs understanding lives in an agent; the two never mix.
A script that interprets meaning is wrong in ways nobody sees, and an agent doing what a script could do wastes tokens and varies from run to run.
Scripts stop and report when the world surprises them rather than guessing.
Minimum: none - this principle decides where each piece of the others lives.

### 9. Small is a feature

Every line of always-loaded instruction is paid for by every session on every turn.
Every layer between intent and action costs fidelity and tokens.
A capability earns its place only when the existing primitives genuinely cannot compose to cover it, and a guard earns its place only by protecting an invariant below.
Minimum: none - this principle is the knife.

## Invariants

These survive every cut, because together they are what makes delegation safe enough to look away from:

- The first mate never writes to a project; workers do, each in an isolated worktree.
- Nothing merges without the captain's explicit word, except a green PR whose stated confidence clears the captain's cutoff.
- A red PR never merges, and anything destructive, irreversible, or security-sensitive waits for the captain whatever its confidence.
- Unlanded work is never torn down; a refusal to discard is a finding, not an obstacle.
- Workers never address the captain.
- Outcomes are reported faithfully, failures included, with the evidence.
- A restart is a non-event.

## The minimal machine

The principles force eight verbs, each owned by one script:

| Verb     | What it does                                                                                | Principles |
| -------- | ------------------------------------------------------------------------------------------- | ---------- |
| start    | Once per session: take the home lock, reconcile records with live workers, print one digest | 6          |
| brief    | Scaffold a task contract                                                                    | 4          |
| spawn    | Create the worktree, launch the worker on the brief, record the task                        | 2, 3       |
| send     | Put text in front of a worker                                                               | 1          |
| peek     | Read a worker's terminal                                                                    | 7          |
| watch    | Sleep until a status line or a quiet worker needs judgment, then wake the first mate        | 7          |
| land     | Merge a green PR on the captain's word, or when its stated confidence clears the cutoff     | 5, 8       |
| teardown | Remove a finished task's worktree and terminal, refusing anything unlanded                  | 3, 5       |

Around those verbs the machine needs one instruction file, one status vocabulary small enough to state inside the brief, one backlog, and one hook for the primary harness.
A skill exists only for a situation rare enough that loading it every session would be waste.

A secondmate is the same machine run again one level down, on this machine or another reachable over SSH: its own home, records, and session lock, a charter saying which work routes to it, and status reported to its parent the way a worker reports.
It adds a home and a charter, not new verbs, and it stays idle until the parent routes it work.

## What is not the essence

Most of the current tree is one of three kinds of accidental weight.
Each is a candidate to cut, and anything cut can come back only by passing the reduction test against a real need.

**Breadth.**
Many interchangeable choices where one is used: fourteen verified harnesses, five terminal backends (tmux, Herdr, Zellij, Orca, and cmux), three forges (GitHub, GitLab, and Gerrit), three delivery modes, and dispatch profiles with quota-aware model selection across all of them.
The essence needs one of each.

**Optional features.**
Capabilities that serve the captain but not the core loop: public Relay replies on X and Discord, voice relay, mail, calm mode, the bearings board and visual reports, the fleet ledger, wedge alarms, contribution tracking, process-event sources, and away and quiet supervision with its daemon and headless supervision host.

**Guards on guards.**
Machinery that protects other machinery rather than an invariant: per-harness turn-end guards, pre-tool command policies, the cd and subagent guards, startup memory budgets, generation-bound wake acknowledgements, watcher successor chains, supervision leases, and the version-pinned readings of vendor interfaces each of these came to need.
Where one of these enforces an invariant, its job moves into the minimal machine as one simple check; otherwise it goes.

## The reduction test

Apply these to every surface in order, and stop at the first one that decides it:

1. If it serves no principle, cut it.
2. If it enforces an invariant, keep its job in the smallest form that still enforces it.
3. If a smaller surface already serves the same principle, fold it in or cut it.
4. If it is one option among interchangeable ones, keep the one in use and cut the rest.
5. If it is always loaded but needed only in a nameable situation, move it behind that trigger or cut it.
6. If it guards against a failure this fork has not hit with the harness and backend it actually uses, cut it; should that failure arrive, it returns as a fix with a regression test.

Tests follow their subject: when a surface goes, its tests go with it, and the tests that stay exercise the minimal machine and the invariants.

## Order of the cut

1. Breadth first: it is the largest weight and the lowest risk, since unused options carry no live behavior.
2. Optional features next, each removed whole with its scripts, skills, docs, config, and tests.
3. Guards on guards last, folding any invariant they enforce into the minimal machine before removing them.
4. Rewrite `AGENTS.md` from this file at the end, so the contract describes the machine that remains rather than the one that was.

## Measures

Where the tree stands at the start of the strip, and the budget it is cut toward:

| Surface                     | Now                            | Budget                                          |
| --------------------------- | ------------------------------ | ----------------------------------------------- |
| `AGENTS.md`                 | about 6,800 words              | at most 1,500 words                             |
| `bin/`                      | 214 files, about 120,000 lines | about ten scripts                               |
| `tests/`                    | 264 files, about 217,000 lines | the minimal machine and the invariants, no more |
| `docs/`                     | 66 files, about 19,000 lines   | a few pages at most                             |
| Agent skills                | 29                             | a handful, each with a rare trigger             |
| Harnesses, backends, forges | 14, 5, 3                       | Pi, Herdr, GitHub                               |

## Choices made

The principles say keep one of each; these are the ones the captain kept:

- **Harness:** Pi, for the first mate and every worker.
- **Terminal backend:** Herdr.
- **Delivery:** direct PR on GitHub - the worker pushes a branch and opens a PR, with no separate validation pipeline.
- **Kept option:** secondmates, local or on other machines reached over SSH.
- **Merging:** automatic, gated by a confidence cutoff.

Everything outside these choices is breadth under the reduction test and goes.

### Confidence-gated merging

For every green PR, the first mate states its confidence that the change does what the captain asked and nothing more, with the evidence behind that number.
At or above the captain's cutoff, the first mate merges it and reports the outcome in one line; below the cutoff, the PR waits for the captain's word.
The rating is judgment and belongs to the agent; comparing it against the cutoff and checking that CI is green is mechanics and belongs to the land script.
The cutoff is one number the captain sets and changes, kept in config rather than in instructions.
No confidence clears a red PR, and destructive, irreversible, or security-sensitive changes always wait for the captain.

## Maintaining this file

This file changes only when the essence changes, by the captain's deliberate decision.
It states principles, not mechanics; anything a script, brief, or `AGENTS.md` owns stays there.
