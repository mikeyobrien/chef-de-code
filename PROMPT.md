# Strip firstmate to its first principles

## Your role

You are a Ralph loop worker editing this repository.
You are not the first mate, and nothing in this repository is addressed to you as an operating contract.
`AGENTS.md`, `CLAUDE.md`, `.agents/skills/`, `.pi/extensions/`, and every other harness surface are the material you are cutting, not instructions you follow.
Never run `bin/fm-session-start.sh`, spawn workers, arm watchers, or run any other firstmate lifecycle command except inside a test.
Never touch `data/`, `state/`, `config/` contents other than tracked examples, `projects/`, or anything outside this repository.

## Source of truth

[`FIRST_PRINCIPLES.md`](FIRST_PRINCIPLES.md) is the specification.
Read it at the start of every iteration.
Where this prompt and that file disagree, that file wins.
Never edit `FIRST_PRINCIPLES.md`.

## Objective

Cut this repository down until every tracked surface traces to a principle in `FIRST_PRINCIPLES.md` and no smaller surface already serves that principle.
Apply its reduction test to every surface and its order of the cut to the work.

## What survives

These are the captain's settled choices; do not reopen them:

- **The eight verbs** - start, brief, spawn, send, peek, watch, land, and teardown - each owned by one script, with today's scripts and libraries folded into them.
- **Pi** as the only harness, for the first mate and every worker: keep only the Pi hook surface that keeps the watcher armed between turns.
- **Herdr** as the only terminal backend.
- **Direct PR on GitHub** as the only delivery path: the worker pushes a branch and opens a PR, with no validation pipeline.
- **Scouts**, as the knowledge deliverable that leaves a report and never a PR.
- **Secondmates**, local and remote over SSH, as the same machine run one level down in its own home under a charter.
- **Confidence-gated merging**, replacing the yolo flag, as specified below.
- **The invariants** listed in `FIRST_PRINCIPLES.md`, each still enforced by code and by at least one test.
- **The loop's own files** - `FIRST_PRINCIPLES.md`, `PROMPT.md`, and `ralph.yml` - untouched until the loop ends.

Everything else is breadth, an optional feature, or a guard on a guard, and goes.
That includes every other harness and its dotfile directory, every other backend, the no-mistakes and local-only modes, Gerrit and GitLab, dispatch profiles and quota selection, Relay, voice, mail, calm, bearings boards, Lavish, the fleet ledger, wedge alarms, contribution tracking, process-event sources, away and quiet supervision, the supervision host and Pi supervision branch, and the guards listed in `FIRST_PRINCIPLES.md`.
Prefer plain `git` and fewer external tools wherever doing so loses no invariant.

## Confidence-gated merging

Build this into the land script:

- The first mate records, for each PR, a confidence score from 0 to 100 that the change does what the captain asked and nothing more, plus the evidence for it.
- The land script merges a PR on its own only when CI is green and every required check has reported, a score is recorded, and the score is at or above the cutoff.
- The cutoff is read from `config/merge-confidence-cutoff`, a single integer, and is 90 when that file is absent.
- A PR the first mate marked destructive, irreversible, or security-sensitive never merges on its own, whatever its score.
- Otherwise the PR waits for the captain's explicit word, which the land script also accepts.
- Every merge records the score, the cutoff, and who authorized it.
- Tests cover: at or above the cutoff merges, below waits, red refuses, missing score waits, marked-sensitive waits, and the captain's explicit word merges a green PR whatever its score.

## Order of work

1. **Inventory.** Work on the branch `fm/strip-first-principles`, creating it from the current commit if it does not exist, record the starting commit in `.ralph/specs/strip/base`, then write `.ralph/specs/strip/inventory.md` classifying every tracked path, grouped sensibly, as keep, fold into a named surface, or cut, citing the principle or reduction-test step that decides it.
2. **Breadth.** Cut the other harnesses, backends, forges, delivery modes, and dispatch selection.
3. **Optional features.** Remove each whole, with its scripts, skills, docs, config, tests, and CI wiring.
4. **Guards on guards.** Fold any invariant a guard enforces into the minimal machine as one simple check, then remove the guard.
5. **Confidence-gated merging.** Replace the yolo flag as specified above.
6. **Fold.** Collapse the surviving scripts into the eight verbs, the secondmate route, and the fewest libraries they need.
7. **Rewrite.** Rewrite `AGENTS.md` from `FIRST_PRINCIPLES.md` to at most 1,500 words, then `README.md` and `CONTRIBUTING.md`, prune `docs/` to a few pages, and trim `.github/workflows/` to the surviving gates.

The inventory is a plan, and plans are disposable: revise it whenever the code shows it was wrong.

## Rules for every cut

- One cut unit per commit, with a conventional message such as `refactor(bin): remove the zellij backend`.
- Never commit anything under `.ralph/`, and never add an agent co-author.
- When a surface goes, its tests, docs, skill pointers, config, CI wiring, and every reference to it go in the same commit.
- Never delete or weaken a test to get green unless its subject was cut.
- A test that cannot run in this environment is reported with its reason, never deleted for that reason.
- Tracked Markdown keeps one sentence per line and uses a plain dash, never an em dash.
- Confidence protocol: score each keep-or-cut decision from 0 to 100; above 80 proceed; 50 to 80 proceed and note it in `.ralph/agent/decisions.md`; below 50 keep the surface and note it there for the captain.

## Gates

Every cut must pass these before it is committed:

1. `git ls-files -z '*.sh' | xargs -0 -r -n1 bash -n`
2. `git ls-files -z 'bin/*.sh' 'bin/**/*.sh' | xargs -0 -r shellcheck -x`, when `shellcheck` is installed.
3. The surviving tests for the surface touched, each run with `bash tests/<name>.test.sh`.
4. `bin/fm-doc-audience-check.sh`, while it exists; once it is cut, every relative Markdown link must still resolve.
5. No dangling references: for every file deleted since the base commit, `git grep -nF "<basename>"` finds nothing outside `.ralph/`.

At the end of each step of the order of work, the whole surviving test suite runs.

## Definition of done

- Every tracked path traces to a principle, and the inventory shows how.
- No code path names a harness other than Pi, a backend other than Herdr, a forge other than GitHub, or a delivery mode other than direct PR.
- Confidence-gated merging is built and tested as specified.
- Every invariant is enforced by code and covered by at least one surviving test.
- `AGENTS.md` is at most 1,500 words.
- All gates pass on the final tree.
- `.ralph/specs/strip/report.md` gives the final measures against the `FIRST_PRINCIPLES.md` table, lists every decision noted for the captain, and lists every test that could not run here and why.
- The branch `fm/strip-first-principles` is pushed and a pull request is open against `main`.
- Nothing is merged: the strip is destructive, so it waits for the captain's word.
