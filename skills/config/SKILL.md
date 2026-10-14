---
name: config
description: Show or change the Agent Crew settings (models per role, base branch, parallelism, board) and where each value comes from. Invoked as /crew:config [key value] [--user].
argument-hint: "[key value] [--user]"
disable-model-invocation: true
allowed-tools: Read, Edit, Write, Bash
---

Show or change the **Agent Crew** configuration.

Arguments: `$ARGUMENTS`

## The levels

Merge these; a later level wins key by key, and `models` merges key by key too:
1. **Kit defaults** (table below).
2. **User level:** `<config dir>/crew.json`. `<config dir>` is `$CLAUDE_CONFIG_DIR` when set, else `~/.claude`. Expand it to an absolute path first: run `echo "$CLAUDE_CONFIG_DIR"` (one Bash call), and if empty run `echo "$HOME"` (another call) and use `<home>/.claude`. The Read tool needs the absolute path.
3. **Project level:** `.claude/crew.json` in the main repo root. It is committed, so it is shared with the team. Inside a worktree, the main repo root is the parent of `git rev-parse --path-format=absolute --git-common-dir`.
4. **A chat override** for one run, and 5. **a plan task's `Model:` line**. These are not stored; mention them in the output only.

Read the files with the Read tool. A missing file is fine; skip it. If a file is not valid JSON, name that file and stop.

## Keys and kit defaults

| Key | Default | Notes |
|---|---|---|
| `project` | main repo's top-level folder name | never a worktree's folder name |
| `board` | none | honoured at **user level** only (see below) |
| `teamBoard` | `false` | project level; set `true` when the whole team writes to one shared board |
| `models.implementer` | `sonnet` | |
| `models.implementer_mechanical` | `haiku` | |
| `models.implementer_hard` | `opus` | |
| `models.reviewer` | `opus` | |
| `models.research` | `sonnet` | |
| `models.follow_up` | `sonnet` | |
| `maxParallel` | `4` | valid range 1-6 |
| `staleMinutes` | `15` | minutes without an update before a running card shows as stale |
| `baseBranch` | the default branch from `git symbolic-ref refs/remotes/origin/HEAD`, else `main` | where part branches start and their PRs (MR on GitLab) target |
| `plansDir` | `docs/superpowers/plans` | |
| `ledger` | `docs/superpowers/ledger.md` | |
| `worktreeSetup` | none | shell command run in each fresh worktree, e.g. `npm ci` |

## Validation

- Model values: `haiku`, `sonnet`, `opus`, `fable`, `inherit`, or a full id starting with `claude-`.
- `maxParallel` is an integer 1-6. `staleMinutes` is a positive integer.
- `board` must start with `https://claude.ai/artifact/`.
- **`board` rule:** a `board` found in the project file is ignored unless that same file sets `teamBoard: true`. When ignored, warn: "board in the project config is ignored (it would publish run data to a shared URL); set it at user level, or set teamBoard: true for a team board".
- `worktreeSetup` is a non-empty command; leave the key out for none.
- **Reviewer floor:** rank `haiku` < `sonnet` < `opus`. If `models.reviewer` resolves to anything weaker than `opus`, or weaker than any implementer tier, or the reviewer or any implementer tier is unrankable (`fable`, `inherit`, a full id), warn that the reviewer should not be weaker than the implementers; a run needs the owner's explicit "yes" and records the downgrade in the ledger.
- **Implementer floor:** if `models.implementer` is `haiku`, warn that plans give decisions, not code, so the default implementer should be `sonnet` or above; Haiku belongs on `implementer_mechanical`. A run needs the owner's explicit "yes".

## With no arguments

Print one table with the columns setting, value and source (kit default, user or project), covering every key. Add any warnings from the validation. Under it, note that the main session's model (brainstorming, specs, plans) is chosen in Claude Code's model picker, not here, and that one run can override a setting in chat.

## With `key value`

Examples: `implementer opus`, `reviewer opus`, `baseBranch develop`, `maxParallel 2`, `board https://claude.ai/artifact/<id> --user`.

1. **Resolve the key.** A role name (`implementer`, `implementer_mechanical`, `implementer_hard`, `reviewer`, `research`, `follow_up`) means `models.<role>`. Other keys are top-level.
2. **Pick the file.** The project file, or the user file when `--user` is given. Create it with only this key if missing. `board` without `--user` is refused: tell the owner to add `--user`, or to set `teamBoard true` first for a team board.
3. **Validate** the value as above; refuse an invalid one and say why.
4. **Edit** with Edit or Write, keeping other keys and formatting.
5. **Show** the changed line before and after, then the merged table. Do not commit.
