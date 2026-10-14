---
name: init
description: Set up the current project for the Agent Crew kit: detect commands, ask the base branch, add the CLAUDE.md section, create .claude/crew.json and ignore .claude/worktrees/. Invoked as /crew:init.
disable-model-invocation: true
allowed-tools: Read, Edit, Write, Glob, Grep, Bash, AskUserQuestion
---

Set up the current project for the Agent Crew kit. Never commit; the owner decides.

This skill's base directory is given at the top of the skill prompt. The plugin root is **two levels above it** (`<base>/../..`). The snippet is `<base>/../../CLAUDE-snippet.md`.

## Steps

1. **Find the main repo root.** If you are in a worktree, use the parent of `git rev-parse --path-format=absolute --git-common-dir`. Work there. If this is not a git repo, say so and stop.
2. **Detect the commands.** Look (Glob/Read) for `package.json` scripts, `Makefile` targets, `pyproject.toml`, `Cargo.toml`, `go.mod`, Gradle (`gradlew`, `build.gradle*`), Xcode (`*.xcodeproj`, `*.xcworkspace`) and SwiftPM (`Package.swift`). From them propose: the unit command, the fast full unit suite, UI tests, lint, a build per target, check-only commands (lint or type-check without writing), parallel-safe test commands (ones that share no ports, devices or database, or "none"), and a worktree setup command (e.g. `npm ci`). Show the table and let the owner confirm or correct it. Anything not detected stays `[fill in]`.
3. **Ask the base branch.** Detect the default with `git symbolic-ref refs/remotes/origin/HEAD` (strip `refs/remotes/origin/`), else `main`. Ask the owner (AskUserQuestion), proposing the detected one: "Part branches will start from, and their PRs (MR on GitLab) will target, which branch?"
4. **CLAUDE.md.** Read the snippet. Take the section below its `---` line. If the project's `CLAUDE.md` already has a "How larger work is planned and run" heading, show the diff you would make and ask before changing it. Otherwise append the section, filling in the confirmed commands and the base branch, and writing `[fill in]` for everything unknown (traps, protected and generated files, never-stage paths, shell and permission rules, invariants). Create `CLAUDE.md` if missing.
5. **Config.** Create `.claude/crew.json` with `project` (the main repo's folder name) and `baseBranch`; keep an existing file's other keys. If `worktreeSetup` was confirmed, add it too. Never put `board` here.
6. **`.gitignore`.** Add `.claude/worktrees/` if absent (create the file if needed).
7. **Offer the board.** Ask whether to run `/crew:board-setup` now (optional; needs a claude.ai sign-in).
8. **Finish.** Run `git status` and show it. List what the owner still has to fill in (every `[fill in]`). Do not stage or commit anything.
