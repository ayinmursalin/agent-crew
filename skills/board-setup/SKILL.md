---
name: board-setup
description: Publish the Agent Board page as a private Artifact with a live database and save its URL to the user-level config. Invoked as /crew:board-setup.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Bash, Artifact, ArtifactData, ToolSearch
---

Publish the Agent Board once; one board serves every project.

This skill's base directory is given at the top of the skill prompt. The plugin root is **two levels above it**. The page is `<base>/../../board/index.html`.

## Steps

1. **Check for an existing board.** Read the user-level config (`$CLAUDE_CONFIG_DIR` else `~/.claude`, file `crew.json`; expand with `echo "$CLAUDE_CONFIG_DIR"` / `echo "$HOME"` to an absolute path first). If it already has a `board`, ask whether to keep it, or publish a new one.
2. **Load the tools** if they are deferred: ToolSearch `select:Artifact,ArtifactData`.
3. **Publish** with the Artifact tool: `file_path` = the absolute path of `board/index.html`, `icon: "progress"`, `capabilities: {"db": {"rules": [{"path": "", "read": "view", "write": "admin"}]}}`, a short `description`. Do not edit the page.
4. **Save** the returned URL as `board` in the user-level config (create the file if missing, keep other keys). Never in the project config.
5. **Verify** with one ArtifactData `list` of `runs` with `query.limit: 1`. An empty result is fine; an error means say so and stop.
6. **Report the link.** It is private until the owner shares it from the page's Share menu (readers can watch; only editors write). Open it while signed in to claude.ai, on the phone too; the live database works only inside the claude.ai viewer. Remind them to share it with teammates deliberately, never as a public link.
