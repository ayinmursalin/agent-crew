# CLAUDE.md snippet

`/crew:init` inserts the section below into the project's `CLAUDE.md` and fills in what it can detect. You can also paste it by hand. Fill in every `[fill in]`: the agents follow whatever this file says, so the more exact the commands, the fewer surprises.

---

## How larger work is planned and run

- Multi-step work follows the **`crew:parallel-parts` skill** on top of **Superpowers**: `superpowers:brainstorming` for the spec, then per part `superpowers:writing-plans` → `superpowers:subagent-driven-development` → `superpowers:finishing-a-development-branch`. One plan and one PR (MR on GitLab) per part, implement → one strong review per task, parallel tasks only in worktrees with separate files, live progress on the Agent Board.
- Tasks go to the **`crew:part-implementer`** agent; reviews go to the **`crew:part-reviewer`** agent (read-only). Paste the task's full text into each dispatch; don't repeat the rules these agents and this file already carry.
- Settings live in `.claude/crew.json` (project, base branch, model per role, parallelism). See them, or change one, with `/crew:config`. The board URL is a user-level setting, not a project one.
- Base branch: `[fill in, e.g. main]`. Each run start confirms it with the owner.
- Plans and the ledger: `[fill in only if not the defaults: docs/superpowers/plans/ and docs/superpowers/ledger.md]`. Each milestone has an overview listing its parts.

### Commands (use exactly these)

- Unit tests (one target): `[fill in]`
- Fast full unit suite (run before every commit): `[fill in]`
- UI / instrumented tests: `[fill in]`
- Lint: `[fill in]`
- Build (each target / form factor): `[fill in]`
- **Check-only commands** the reviewer may run (they never write or fix anything): `[fill in, e.g. lint without --fix, type-check, the test suites]`
- **Parallel-safe test commands** (safe to run in several worktrees at once: no shared ports, devices or database): `[fill in, or "none"]`
- **Worktree setup**, run in each fresh worktree before work starts: `[fill in, e.g. npm ci, or "none"]`

### Known traps

- `[fill in: things that look right but break, flaky tests, tools that must not be re-run, order-dependent steps]`

### Protected and generated files

- Never hand-edit generated files: `[fill in paths and the command that regenerates each]`
- Protected files (secrets, keystores, credentials): `[fill in]`. Agents never read or print them.

### Never stage

- The ledger and plans (the controller owns them), plus `[fill in: build output, local config, caches]`.

### Shell and permission rules for agents

- `[fill in, e.g. one command per Bash call, no chained commands, use the Read/Edit/Write tools instead of cat/sed/heredocs, which commands are allow-listed]`

### Never break

- `[fill in: invariant 1, e.g. shared database fields, stable keys, analytics names]`
- `[fill in: invariant 2, e.g. never fake a paid state, even in debug builds]`
- Never wipe devices, uninstall the app or erase test data without asking.
