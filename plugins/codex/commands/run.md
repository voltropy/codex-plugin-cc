---
description: Tell Codex to do something — runs with full bash, git, web, and file permissions
argument-hint: '[--wait|--background] [--model <model>] [--effort <level>] <prompt>'
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(node:*), Bash(git:*), AskUserQuestion
---

Run a Codex task with full permissions: bash commands, git, web search and fetch, and read/write/edit access to files in the workspace.

Raw slash-command arguments:
`$ARGUMENTS`

Core constraint:
- Your only job is to launch the run and return Codex's output verbatim to the user.
- Do not modify the user's prompt, add constraints, or rewrite their intent.

Execution mode rules:
- If the raw arguments include `--wait`, do not ask. Run in the foreground.
- If the raw arguments include `--background`, do not ask. Run in a Claude background task.
- Otherwise, ask once using `AskUserQuestion` with two options, recommending background:
  - `Run in background (Recommended)`
  - `Wait for results`

Argument handling:
- Preserve the user's arguments exactly.
- Do not strip `--wait` or `--background` yourself.
- The companion script parses `--wait` and `--background`, but Claude Code's `Bash(..., run_in_background: true)` is what actually detaches the run.
- Supported flags: `--model <model>` (or `-m`), `--effort <none|minimal|low|medium|high|xhigh>`, `--resume-last` / `--resume`, `--fresh`.

Foreground flow:
- Run:
```bash
node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" run "$ARGUMENTS"
```
- Return the command stdout verbatim, exactly as-is.
- Do not paraphrase, summarize, or add commentary before or after it.

Background flow:
- Launch with `Bash` in the background:
```typescript
Bash({
  command: `node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" run "$ARGUMENTS"`,
  description: "Codex run",
  run_in_background: true
})
```
- Do not call `BashOutput` or wait for completion in this turn.
- After launching, tell the user: "Codex run started in the background. Check `/codex:status` for progress."
