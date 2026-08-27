# Handoff: Chinese Learning MVP

**Work Item ID**: `001-chinese-learning-mvp`
**Updated**: `2026-08-25 21:34 +07`
**Status**: Blocked

## Current Goal

Dispatch the reviewed `backend-foundation` OpenCode packet as the first backend
foundation deliverable for the SelfHan Chinese Learning MVP.

## Current Phase

Implement

## What Is Done

- Restored the backend checkout to its pre-task state after the incorrect direct
  implementation attempt.
- Created the full MVP specification, data model, API proposal, staged plan, and
  traceable task list under `specs/001-chinese-learning-mvp/`.
- Created and validated the first OpenCode packet at
  `runs/001-auth-content-foundation/workers/backend-foundation/`.
- Fixed the dispatcher AppleScript closing syntax in
  `scripts/iterm2-runtime.applescript`.

## Current State

- Branch/worktree: No worker worktree exists; dispatch did not reach worktree
  creation.
- Important files changed: `scripts/iterm2-runtime.applescript`,
  `specs/001-chinese-learning-mvp/`, and `runs/001-auth-content-foundation/`.
- User decisions captured: Must-have is the core MVP; should-have is staged after
  core; AI, community, recognition, leaderboard, and HSK 4-6 remain excluded.
- Persona lens decisions to preserve: Publish and approve each backend contract
  before dispatching its dependent frontend worker; one worker per repository per
  run owns one isolated branch.

## Next Action

After iTerm2 is available, run:

```sh
sh scripts/setup-opencode-project.sh --detach --session selfhan-mvp --repos opencode-repos.tsv
```

Then dispatch only the prepared backend packet:

```sh
sh scripts/dispatch-opencode-workers.sh --detach --session selfhan-mvp --repos opencode-repos.tsv runs/001-auth-content-foundation/workers/backend-foundation
```

## Open Questions / Blockers

- The managed dispatcher requires iTerm2 with bundle ID `com.googlecode.iterm2`.
  It is not installed or not discoverable in the current desktop environment.
- HSK 1-3 expansion still needs a licensed or team-authored content/audio source;
  this does not block the demo foundation packet.

## Files To Read First

- `AGENTS.md`
- `.agent/routes/00-core.md`
- `.agent/routes/09-handoff.md`
- `specs/001-chinese-learning-mvp/handoff.md`
- `specs/001-chinese-learning-mvp/tasks.md`
- `runs/001-auth-content-foundation/workers/backend-foundation/prompt.md`

## Verification So Far

| Command / Check | Result | Notes |
| --- | --- | --- |
| `sh scripts/update-opencode-task-status.sh --packet runs/001-auth-content-foundation/workers/backend-foundation --validate` | Pass | The worker task table is harness-valid. |
| `git diff --check` | Pass | No whitespace errors in control-repository changes. |
| Markdown review checklist | Pass | Each created Markdown file has one title, valid heading order, blank-line separation, no trailing whitespace, and one final newline. |
| `osascript -e 'using terms from application "Finder"' -e 'end using terms'` | Fail as expected | Reproduces AppleScript syntax error `-2741` for the old closing form. |
| `osascript -e 'using terms from application "Finder"' -e 'end using terms from'` | Pass | No AppleScript syntax error; desktop connection warnings are environmental. |
| `sh scripts/setup-opencode-project.sh --detach --session selfhan-mvp --repos opencode-repos.tsv` | Blocked | Script now reaches iTerm2 lookup and fails with bundle ID unavailable `-1728`. |

## Debug Notes

| Symptom | Cause | Fix / Workaround | Verified By | Avoid Next Time |
| --- | --- | --- | --- | --- |
| Runtime setup reported AppleScript error `-2741` | `iterm2-runtime.applescript` ended a `using terms from` block with `end using terms` | Use `end using terms from` | Minimal `osascript` reproduction with invalid and valid closing forms | Compile or lint the AppleScript before invoking the dispatcher |
| Runtime setup reported AppleScript error `-1728` | iTerm2 bundle ID `com.googlecode.iterm2` is unavailable | Install or expose iTerm2, then run managed setup | Second setup attempt reached application lookup | Check runtime dependency before preparing a release dispatch |

## Resume Prompt

```text
Resume work item 001-chinese-learning-mvp.
Read AGENTS.md, .agent/routes/00-core.md, .agent/routes/09-handoff.md,
specs/001-chinese-learning-mvp/handoff.md, and specs/001-chinese-learning-mvp/tasks.md.
Verify current files before editing. If iTerm2 is available, run the documented
setup command and dispatch only runs/001-auth-content-foundation/workers/backend-foundation.
Do not edit frontend or backend directly from source-trust.
```
