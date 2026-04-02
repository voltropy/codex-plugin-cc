---
name: codex-run-runtime
description: Internal helper contract for invoking the codex-companion run subcommand with full permissions
user-invocable: false
---

# Codex Run Runtime

Use this skill only inside the `codex:codex-run` subagent.

Primary helper:
- `node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" run "<raw arguments>"`

Execution rules:
- The run subagent is a thin forwarder. Its only job is to invoke `run` once and return stdout unchanged.
- `run` uses `danger-full-access` sandbox automatically. Do not add `--write`; it has no effect here.
- Prefer the helper over hand-rolled `git`, direct Codex CLI strings, or any other Bash activity.
- Do not call `setup`, `review`, `adversarial-review`, `task`, `status`, `result`, or `cancel` from `codex:codex-run`.
- You may use the `gpt-5-4-prompting` skill to tighten the user's request into a better Codex prompt before the single `run` call.
- That prompt drafting is the only Claude-side work allowed. Do not inspect the repo, solve the task yourself, or add independent analysis outside the forwarded prompt text.
- Leave `--effort` unset unless the user explicitly requests a specific effort.
- Leave model unset by default. Add `--model` only when the user explicitly asks for one.
- Map `spark` to `--model gpt-5.3-codex-spark`.

Command selection:
- Use exactly one `run` invocation per handoff.
- If the forwarded request includes `--background` or `--wait`, treat that as Claude-side execution control only. Strip it before calling `run`, and do not treat it as part of the natural-language task text.
- If the forwarded request includes `--model`, normalize `spark` to `gpt-5.3-codex-spark` and pass it through to `run`.
- If the forwarded request includes `--effort`, pass it through to `run`.
- If the forwarded request includes `--resume`, strip that token from the task text and add `--resume-last`.
- If the forwarded request includes `--fresh`, strip that token from the task text and do not add `--resume-last`.
- `--resume`: always use `run --resume-last`, even if the request text is ambiguous.
- `--fresh`: always use a fresh `run` invocation, even if the request sounds like a follow-up.

Safety rules:
- Preserve the user's task text as-is apart from stripping routing flags.
- Do not inspect the repository, read files, grep, monitor progress, poll status, fetch results, cancel jobs, summarize output, or do any follow-up work of your own.
- Return the stdout of the `run` command exactly as-is.
- If the Bash call fails or Codex cannot be invoked, return nothing.
