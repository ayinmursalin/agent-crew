# Changelog

## 1.5.0

- **Parallel tasks are scheduled from a dependency DAG, not "prep, then all at once".** The old shape (one sequential prep task, then every other task in parallel) only compared each task with the prep, so it hid parallelism. A task that depended only on the first task still waited for the prep task, because nobody compared the two directly.
  - `crew:workflow` §4: every plan has a **dependency matrix** that classifies every pair of tasks as independent, needs-code, file-overlap, test-infra-overlap or build-only (never an edge). Every edge names a file or symbol, or it is removed. The plan tries to cut edges (move a shared edit or seam into the smallest earlier task, prefer a task-local fake, give exact seams) and records each cut. Four new self-review checks: every pair classified, every edge has a cause, no needless waits, the longest chain as short as the plan allows.
  - §8: a task is **ready** once every task it depends on has landed (committed on the part branch, merged if it ran in a worktree, and reviewed with no open Critical or Important finding on the seam it uses). The controller dispatches every ready task at once as a rolling queue up to `maxParallel`, and re-checks readiness each time a task lands. One task may run in the main checkout while others run in worktrees.
  - Each worktree task's base is the part-branch commit that contains all its dependencies, not a fixed prep SHA. The worktree first steps are unchanged otherwise.
  - Optional **early start:** a dependent may start before its dependency's review when the seam it uses is verbatim in the plan; if a fix changes that seam, the dependent is rebased onto the new base before its merge and review.
  - Safety rules kept: disjoint files per running task, a build or derived-data folder per worktree (now stated explicitly), never resume a returned worktree agent, `--no-ff` merges, review after the merge, and Critical or Important fixes before any dependent using the affected seam.
  - §5 computes the schedule at run start and shows the owner the DAG. §6: the ledger's `Dependencies:` line becomes `Edges:` (Ti → Tj: cause) plus a `Start order:` line; each task row notes its base and which tasks it unblocked. Parallel groups are called ready sets or DAG levels, never waves.
  - A worked example (an illustrative 6-task part, any stack: matrix, cuts, DAG and schedule) is in §8.
  - `crew:implementer`: worktree prompts give a base SHA instead of a prep SHA, and builds go to a folder inside the worktree. `crew:reviewer`: checks seams other tasks use against the plan, and names the seam each Critical or Important issue touches.
- **Board: times, type filter and task sort.**
  - Each card shows when the agent started and finished and how long it took ("Took 13 min"), or "Running 20 min" while it works. The run summary does the same for the whole run.
  - New optional fields: `finishedAt` on agent rows (set by the agent's last write and by the controller's finalize) and on the run row (set when the run is marked done). `startedAt` on an agent row is now set when it switches to `running`, not when the queued row is created. Older rows without `finishedAt` fall back to their last update.
  - **Type** filter: All types, or only Implementer, Reviewer, Follow-up or Research (only the types the run has).
  - **Sort:** "By status" (as before) or "By task": research, then each task's rows in the order they happen (T1 implementer, T1 reviewer, T1 fix, T1 re-review, T2 implementer…), then follow-ups and the final review.
  - Fixed: the "No runs yet" panel and the Project and Run pickers showed even when they should be hidden.
- **Upgrade:** `claude plugin update crew@agent-crew`, and republish `board/index.html` to your existing board URL (Artifact publish with that `url`, so the link stays the same). A run already in progress can finish under its old plan; write the matrix in the next part's plan.

## 1.4.0

- **Resolved reviews no longer stay Blocked.** A review that said `❌ changes needed` used to count as Blocked forever, even after the fix was re-reviewed and approved, so a finished run could show "3 Blocked".
  - Agent rows have a new optional field, `resolvedBy`: the agentId of the re-review that closed the review's findings.
  - The board shows such a row with a grey "Resolved by <id>" pill, a neutral stripe, and the original verdict in small text. It counts as Done in the tally and the overall progress, and "Running only" hides it.
  - `crew:workflow` §7: when a re-review of a fix passes (`✅ ready`, or `approve with follow-ups` with no Critical or Important left), the controller sets `resolvedBy` on the original review row and on any earlier re-review that asked for changes, pinned with `if_version`.
- **The controller finalizes every board row.** Agents that skipped their own board writes left rows at "running, step 0", holding the overall progress down after the work was done.
  - `crew:workflow` §7: as soon as an agent's report arrives, before anything else, the controller sets the row's final status, `step = steps`, `stepLabel`, and the commit, tests or verdict from the report, whether or not the agent wrote its row. §8 does the same for each returning parallel agent.
  - §10: before marking the run done, the controller queries all of the run's rows and finalizes any that are not done, blocked or failed.
  - `crew:implementer` and `crew:reviewer`: the board stays best-effort, but a deferred `ArtifactData` tool is loaded first, never a reason to skip.
- **Upgrade:** `claude plugin update crew@agent-crew` in each project, and republish `board/index.html` to your existing board URL (Artifact publish with that `url`, so the link stays the same) so it renders resolved reviews. Old rows need no change.

## 1.3.0

- **Choose your Superpowers:** two ways to install.
  - `crew-full@agent-crew` (new bundle) installs crew plus `superpowers@agent-crew` 7.0.0, built from source ([obra/superpowers](https://github.com/obra/superpowers) tag `v7.0.0`, pinned to commit `bb92a77`). Use it if you don't have Superpowers or want the tested version.
  - `crew@agent-crew` no longer declares a Superpowers dependency. It uses whichever copy you have, e.g. `superpowers@claude-plugins-official`. `allowCrossMarketplaceDependenciesOn` is removed.
- **Design always writes a spec:** superpowers 7's brainstorming writes a design document only for work it sizes as a project. `crew:workflow` now tells it the size is "project", lets the owner drop crew when the work is really small, and still writes the spec when the owner skips the questions.
- **Plans record decisions, not code,** matching writing-plans since superpowers 6.4.2: exact seams between tasks, test names and assertions, and code only where the code is the decision. `Model: mechanical` tasks still get full code. The implementer and reviewer check against those decisions.
- **Implementer floor:** a default implementer of `haiku` (config or chat override) now gets a warning at run start and needs the owner's "yes", since plans no longer carry code. `implementer_mechanical` stays `haiku`. Examples in the README and `/crew:config` no longer suggest Haiku as the default implementer.
- **`/crew:init`** checks Superpowers first: missing, two copies, or older than 5.1.0.
- **Upgrade:** nothing to do if you keep your current Superpowers; updating `crew` leaves it installed. To switch to 7.0.0, uninstall `superpowers@claude-plugins-official` and run `/plugin install crew-full@agent-crew`.
- **Bumping the pinned Superpowers:** change `ref`, `sha` and `version` of the `superpowers` entry in `.claude-plugin/marketplace.json` (`git ls-remote --tags https://github.com/obra/superpowers` gives the commit).

## 1.2.0

- **Breaking rename:** agents `crew:part-implementer` → `crew:implementer` and `crew:part-reviewer` → `crew:reviewer`; skill `crew:parallel-parts` → `crew:workflow`. The plugin prefix already says "crew", so the old names repeated themselves. "Part" stays the name of the unit of work (one plan, one PR).
- **Upgrade:** in each project's `CLAUDE.md`, replace those three names (or re-run `/crew:init` and accept the diff).

## 1.1.0

- **Name:** first public release as **Agent Crew** (repo and marketplace `agent-crew`, plugin `crew`), MIT licensed. Commands are `/crew:*`, agents `crew:part-implementer` / `crew:part-reviewer`, config `.claude/crew.json`.
- **Packaging:** declares `superpowers` (from `claude-plugins-official`) as a dependency; the marketplace allows that cross-marketplace dependency. Tested with superpowers 5.1.0.
- **Command skills:** `/crew:config` (now a skill; `commands/config.md` removed), `/crew:init`, `/crew:board-setup`, `/crew:board-prune`.
- **Init** asks for the base branch (proposing the repo's default), detects test, lint and build commands, inserts the CLAUDE.md section, creates `.claude/crew.json` and ignores `.claude/worktrees/`.
- **Config:** new keys `baseBranch`, `staleMinutes` (15), `plansDir`, `ledger`, `worktreeSetup`, `teamBoard`; `maxParallel` default 4 (range 1-6); `$CLAUDE_CONFIG_DIR` honoured at user level; `board` is user-level only unless `teamBoard: true`; value validation and a reviewer-floor warning.
- **Base branch:** each run start confirms it with the owner; it is never assumed.
- **Parallel work:** agents prove they are in a worktree, then `git reset --keep <prep-sha>` (worktrees are created from the default branch); returned parallel agents are re-dispatched, never resumed; merges use `--no-ff` and review happens after the merge.
- **Board:** run and agent ids, `-r2` retries, controller finalizes rows, board probe at run start, pruning of old runs. The page subscribes per run, keeps open logs open, shows honest overall progress from `run.tasks`, stale/idle/abandoned states from `run.staleMinutes` against the viewer's clock, verdict colours, and a "Running only" filter.
- **Agents:** tool allowlists, inline TDD rules (quote the red failure), reviewer check-only commands and per-check report headings.
- **CLAUDE snippet:** namespaced skill and agent names, plus sections for commands, traps, protected files, never-stage paths, shell rules and invariants.

## 1.0.0

- Initial kit: `parallel-parts` skill, `part-implementer` and `part-reviewer` agents, `/crew:config` command, Agent Board page, CLAUDE.md snippet and plugin manifests.
