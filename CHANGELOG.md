# Changelog

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
