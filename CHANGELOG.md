# Changelog

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
