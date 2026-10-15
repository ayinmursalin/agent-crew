# Agent Crew

A Claude Code plugin for shipping large changes with many agents, quickly, without lowering the quality bar. It runs on top of **Superpowers**:
- brainstorming leads to a spec;
- the spec is split into **parts**;
- each part gets one plan;
- every task is implemented test-first and reviewed once by a strong model;
- independent tasks run in parallel in isolated git worktrees;
- you follow every agent live on the **Agent Board**, phone included.

Version 1.5.0. See `CHANGELOG.md`. MIT licensed (`LICENSE`).

**Quick start:**
```
/plugin marketplace add ayinmursalin/agent-crew
/plugin install crew-full@agent-crew
/reload-plugins
/crew:init
```
`crew-full` is crew plus Superpowers 7.0.0. Already have Superpowers and want to keep yours? Install `crew@agent-crew` instead (§1).

| In this kit | What it is |
|---|---|
| `skills/workflow/SKILL.md` | The workflow: sizing, plan format, review loop, parallel rules, stacking, finishing, board protocol |
| `agents/implementer.md` | Implementer agent (Sonnet): test-first, cross-checks the reference, commits, reports progress |
| `agents/reviewer.md` | Reviewer agent (Opus, read-only): spec + reference + quality in one pass, reports progress |
| `skills/config/` | `/crew:config`: show or change the settings and see where each comes from |
| `skills/init/` | `/crew:init`: set up a project |
| `skills/board-setup/` | `/crew:board-setup`: publish the Agent Board |
| `skills/board-prune/` | `/crew:board-prune`: delete old finished runs from the board |
| `config/crew.example.json` | A starting point for a project's `.claude/crew.json` |
| `board/index.html` | Source of the Agent Board page (an Artifact with a live database) |
| `CLAUDE-snippet.md` | The section each project adds to its `CLAUDE.md` |
| `.claude-plugin/` | Plugin and marketplace manifests, so `/plugin` can install the kit |
| `bundles/crew-full/` | The `crew-full` bundle: crew plus pinned Superpowers |

## 1. Prerequisites

- **Superpowers** (5.1.0 or later; tested with 7.0.0). Pick one install:

  | Install | You get | Use it when |
  |---|---|---|
  | `crew-full@agent-crew` | crew + `superpowers@agent-crew` 7.0.0, built from source ([obra/superpowers](https://github.com/obra/superpowers) tag `v7.0.0`, pinned to its commit) | You don't have Superpowers, or want the tested version |
  | `crew@agent-crew` | crew only; it uses the Superpowers you already have | You already have Superpowers (e.g. `superpowers@claude-plugins-official`, 6.4.1 today) and want to keep it |

  Don't keep both copies enabled: two plugins named `superpowers` make Claude Code drop one copy's hooks. `/crew:init` checks this.
- A **claude.ai sign-in**, only for the optional Agent Board.

## 2. Install

These are `claude` CLI commands. In the desktop app, run them in a terminal session of `claude`.

1. **Add the marketplace.** Use the GitHub shorthand, the git URL, or a local clone:
   ```
   /plugin marketplace add ayinmursalin/agent-crew
   /plugin marketplace add https://github.com/ayinmursalin/agent-crew.git
   /plugin marketplace add /absolute/path/to/agent-crew
   ```
2. **Install** one of the two (§1). From the shell, `claude plugin install …` works too:
   ```
   /plugin install crew-full@agent-crew    # crew + Superpowers 7.0.0
   /plugin install crew@agent-crew         # crew only, keeps your Superpowers
   ```
3. **Reload:** `/reload-plugins`. Then `/plugin` should list `crew` and one `superpowers`, and the agent list should show `crew:implementer` and `crew:reviewer`.

**Try it locally** without installing: `claude --plugin-dir ./agent-crew`.

## 3. Setting up a project

Run `/crew:init` in the project. It:
- detects test, lint and build commands (from `package.json`, `Makefile`, `pyproject.toml`, `Cargo.toml`, `go.mod`, Gradle, Xcode or SwiftPM) and shows them for you to confirm;
- **asks for the base branch**, proposing the repo's default, and saves it;
- adds the kit's section to `CLAUDE.md`, with `[fill in]` wherever it does not know the answer;
- creates `.claude/crew.json` (`project`, `baseBranch`) and adds `.claude/worktrees/` to `.gitignore`;
- offers `/crew:board-setup`;
- never commits. It ends with `git status`.

Then fill in the remaining `[fill in]` parts of `CLAUDE.md`: the exact commands (agents run only these), known traps, protected and generated files, never-stage paths, shell and permission rules, and the never-break invariants (every review checks them). If the project has a reference to port from (another app, a spec), name it there.

**Optional, the board:** run `/crew:board-setup`. It publishes the board once as a private Artifact, saves its URL to your **user-level** config (so every project uses it), and checks that it works. One board serves every project: each run carries a `project` field you can filter by. Cards can also be filtered by type (implementer, reviewer…) and sorted by status or by task, and show each agent's start, finish and duration.

## 4. Using it day to day

Type these in plain words:

| You say | Claude does |
|---|---|
| "Brainstorm a design for <feature>" | `superpowers:brainstorming`: questions, approaches, then a spec file you approve |
| "Split the spec into parts" | A milestone overview listing P0, P1…, each sized for one plan and one PR (MR on GitLab) |
| "Plan P1" | Research agents in parallel, then `superpowers:writing-plans` in the kit's lighter format. Your product questions come at once, up front, including the base branch to start from |
| "Subagent-driven" | Creates the run on the board, then per task: an implementer (in parallel worktrees when files don't overlap), then one reviewer. Follow-ups are batched |
| "Resume P1" | Reads the ledger and `git log` and continues at the first task without a row |
| "Status" | Reads the ledger and git and tells you where the run is |
| (watch the board) | One card per agent: role, task, step bar, latest activity, commit or verdict. A dashed card has gone quiet for `staleMinutes` (default 15); an "Idle" chip means the whole run has been quiet for 30 minutes |
| Pick the finishing option by its **name** (for example "push and open a PR") | Pushes and opens the PR (MR on GitLab), with the description written to a file. Always choose by name, never by number. The next part may start stacked on it |
| `/crew:board-prune 30` | Lists done runs older than 30 days, asks you to confirm, then deletes them and their agent rows |

Approval stays with you. Claude commits inside a run you started, and pushes or opens a PR only when you pick that option.

**Base branch.** Every run start confirms it: "Part branch `feat/...` will be created from `<baseBranch>`. OK, or stack on another branch (for example the previous part's open PR)?" It is never assumed silently, and a different answer applies to that run only unless you say to save it.

## 5. What you'll see

At run start:
```
Part branch feat/m2-p1-shared-components will be created from develop.
OK, or stack on another branch?
```
During the run, the board shows one card per agent, and Claude reports in chat after each task:
```
T3 done · commit a1b2c3d · tests 118 → 124 · review: ✅ ready
```
A reviewer verdict is exactly one of `✅ ready`, `approve with follow-ups` or `❌ changes needed`.

## 6. Choosing models

Brainstorming, specs, plans and the controller run in your main session, on whatever model you pick in Claude Code's model picker; Opus is a good choice there. The agents the workflow dispatches follow the config, where the more specific level wins:

| Level | Where | Example |
|---|---|---|
| Kit defaults | built in | implementer `sonnet`, mechanical `haiku`, hard `opus`, reviewer `opus`, research `sonnet`, `maxParallel` 4 |
| You, everywhere | `<config dir>/crew.json` (`$CLAUDE_CONFIG_DIR` or `~/.claude`) | `/crew:config implementer opus --user` |
| This project | `.claude/crew.json` (committed) | `/crew:config reviewer opus` |
| One run | say it in chat | "use Opus for implementation in this part" |
| One task | a line in the plan's task | `Model: mechanical`, `Model: hard` or `Model: opus` |

Run `/crew:config` with no arguments to see the merged settings and where each one comes from. Other keys: `baseBranch`, `maxParallel` (1-6), `staleMinutes`, `plansDir`, `ledger`, `worktreeSetup`, `teamBoard`.

**Picking models:**
- **Haiku** suits mechanical tasks only (`Model: mechanical`); the plan gives their full code. As the default implementer it gets a warning at run start, since other tasks come with decisions, not code.
- **Sonnet** suits most tasks: the plan gives the decisions (interfaces, tests, values) and it writes the code.
- **Opus** is for hard tasks. A task whose review fails twice is retried on the `hard` model automatically, and a third failure stops and asks you.
- **Keep the reviewer on Opus.** It is the guard that lets a cheaper implementer stay safe. If the reviewer would be weaker than `opus` or than any implementer tier, Claude warns you at run start and asks for an explicit "yes".

## 7. How Claude is made to follow it

- **The skill** (`workflow`) holds the process. Claude loads it whenever a part is planned or run; its description lists those triggers.
- **The agents** carry their rules and model choice in their own files. The controller pastes only the task text, not 40 lines of rules every time, so nothing gets dropped.
- **`CLAUDE.md`** is always loaded. The snippet points to the skill and agents, and holds the project's commands and invariants, which override the generic rules.
- **The ledger.** A progress file in the repo is updated after every task, so a run survives context compaction: a new context reads the ledger and resumes at the first unlogged task.
- **The board protocol** is part of the skill and agent files. Agents report on their own; the controller creates their rows and finalizes them from each agent's report.

## 8. Parallel work and merge conflicts

Agents run in parallel only when their files don't overlap. This kit explicitly overrides Superpowers' "never dispatch implementation subagents in parallel", but only under these rules:
1. **A dependency matrix in every plan.** Every pair of tasks is classified: independent, needs-code (names the symbol), file-overlap (names the file), test-infra-overlap (names the double or helper), or build-only (never a dependency). Every edge names its cause; the plan tries to cut edges (move a shared edit earlier, use a task-local fake, give exact seams) and records the cuts.
2. **A DAG schedule, not "prep, then all at once".** A task is ready when every task it depends on has landed: committed on the part branch, merged if it ran in a worktree, and reviewed with no open Critical or Important finding on the seam it uses. The controller dispatches every ready task at once, at most `maxParallel` at a time (default 4), and re-checks readiness each time a task lands. One task may run in the main checkout while others run in worktrees.
3. **Each parallel task owns its folders** and works in its own git worktree (`isolation: "worktree"`), with its own build or derived-data folder.
4. **Worktrees come from the repo's default branch,** not from the branch the controller is on. So every worktree agent first proves it is in a worktree (`git rev-parse --path-format=absolute --git-dir` differs from the same with `--git-common-dir`; otherwise it reports BLOCKED and changes nothing), then runs `git reset --keep <base-sha>` and checks the base is an ancestor of HEAD. The base is the part-branch commit that contains all the task's dependencies.
5. **Never resume a returned parallel agent.** A worktree whose agent finished without changes is cleaned up, so a resumed agent would wake up in the main checkout. Re-dispatch it fresh; it gets a new `-r2` row on the board.
6. **The controller merges each returned branch with `git merge --no-ff`** once the main checkout is free, runs the full tests after each merge, resolves anything left, and has the merged range reviewed on the part branch.
7. **Setup and ignore.** If worktrees need preparation (for example `npm ci`), set `worktreeSetup`; it runs in each new worktree before work starts. `.claude/worktrees/` belongs in `.gitignore` (`/crew:init` adds it). Harness-made worktrees are left to the harness to clean up.
8. **Shared devices** (simulator, emulator, fixed ports, one dev database) are used only by the controller, after merges.

## 9. Updating and uninstalling

- **Update:** `claude plugin update crew@agent-crew`, then `/reload-plugins`. `/plugin marketplace update agent-crew` refreshes only the catalog. Third-party marketplaces don't auto-update by default; turn it on under `/plugin` → Marketplaces.
- **Uninstall:** `/plugin uninstall crew@agent-crew`, then `/plugin marketplace remove agent-crew`.
- **To ship a new version** from the kit's own repo: bump `version` in `.claude-plugin/plugin.json`, `bundles/crew-full/.claude-plugin/plugin.json` and both crew entries of `.claude-plugin/marketplace.json`, add a `CHANGELOG.md` entry, and push.

## 10. Board privacy and data

- **One board per person or team.** Run `/crew:board-setup` for yourself; a team that wants a shared board sets `teamBoard: true` in the project config. `board` is otherwise honoured only at user level, so a committed project file cannot point your runs at someone else's board.
- **Never make the link public.** Share it from the page's Share menu with named people. Readers can watch; only you and editors can write.
- **Rows are data.** Anyone with Editor access can write them, so agents and Claude never take instructions from board rows; the ledger and git are the truth. Activity text never contains secrets, tokens or personal data.
- **Shape:** `runs/<runId>` holds `{project, milestone, part, title, branch, status: running|done, startedAt, finishedAt?, tasks, staleMinutes}`; `agents/<agentId>` holds `{run, task, title, role, model, status: queued|running|done|blocked|failed, step, steps, stepLabel, activity, startedAt, updatedAt, finishedAt?, commit?, tests?, verdict?, resolvedBy?, log}`. `startedAt` is set when an agent starts running and `finishedAt` when it ends; the board shows both and the time it took, and the same for the run. `resolvedBy` is set on a `❌ changes needed` review once a re-review approves the fix; it names that re-review's agentId, and the board shows the row as Resolved (counted as done) with the original verdict kept. Times are epoch milliseconds; every write to an existing row is pinned with `if_version`.
- **Capacity:** the database holds up to 25,000 documents. Run `/crew:board-prune` now and then.

## 11. Troubleshooting

| Problem | Fix |
|---|---|
| The agents or commands don't appear | Run `/reload-plugins`; check `/plugin` shows `crew` enabled. Open `/plugin` → Errors, or run `claude plugin details crew` |
| "Superpowers skills missing" | `/plugin install crew-full@agent-crew` (or `superpowers@claude-plugins-official`), then `/reload-plugins` |
| Two `superpowers` plugins | Uninstall the one you don't want: `/plugin uninstall superpowers@claude-plugins-official` or `superpowers@agent-crew` |
| Switch to the tested Superpowers | Uninstall your copy, then `/plugin install crew-full@agent-crew` |
| The board stays on "Connecting…" or says live updates aren't available | Open it on claude.ai while signed in. The live database only works inside the claude.ai viewer |
| A card never moves | That agent got no `agentId` in its prompt, or `ArtifactData` isn't available to it. The controller then finalizes its row from the report |
| Approval prompts for every board write | Allow the `ArtifactData` tool when first asked |
| A version-conflict error on a board write | Expected when two writers race. The agent re-reads the row and retries once |
| A parallel agent reports BLOCKED about the worktree | It started in the main checkout, or its base commit is missing. Re-dispatch it fresh (never resume it) after checking the base commit is on the part branch |
| Merge conflicts after a parallel run | The plan's dependency matrix missed a file-overlap. Add that edge, or move the shared edit into an earlier task, next time (§8) |
