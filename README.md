# Docwatch

A Hermes Agent that keeps a team's docs in sync with the product.

Engineers ship changes fast, and the docs other teams depend on (setup guides, API examples, sales demo scripts) quietly go stale. Nobody notices until a customer or a new hire hits the broken step. Docwatch re-tests those docs against the live app every time code changes and tells the doc owner exactly what broke and how to fix it.

Built on [Hermes Agent](https://github.com/NousResearch/hermes-agent) by Nous Research.

## How it works

1. An engineer commits and pushes a change.
2. A git hook triggers a scheduled Hermes job (there is also a weekly safety net run).
3. Docwatch clones the repo fresh, the way a new hire would, starts the app and runs every step in every doc exactly as written.
4. When a step fails, it reads the latest diff to find the cause, writes a corrected command and runs it against the live app to prove the fix works.
5. It opens a pull request with the fix and alerts the doc owner's team channel. Merging the PR is the approval.

If everything still works, it stays silent.

## Design decisions

**Test like a new hire.** Every run starts from a fresh clone, so leftovers from earlier runs never cause false alarms.

**Environment noise is not drift.** A busy port or a slow server start is retried, not reported. A watchdog that cries wolf gets ignored.

**Verified fixes only.** Every suggested fix is executed against the running app before it is proposed.

**Humans approve.** Docwatch never pushes to main and never merges. It never touches application code, only docs.

**Professional output.** No emojis or filler in alerts, PR titles or PR bodies.

## Demo

The demo company lives at [leadly-hq/leadly](https://github.com/leadly-hq/leadly). Its docs are owned by a Tech team and a GTM team (see `docs/owners.yaml`). You can see the pull requests Docwatch opened after breaking changes [here](https://github.com/leadly-hq/leadly/pulls?q=is%3Apr).

## Files

`skill/SKILL.md` is the Hermes skill.
`hooks/post-commit` is the git hook that triggers a run after each commit.

## Setup

1. Copy `skill/SKILL.md` to `~/.hermes/skills/devops/docwatch/SKILL.md`.
2. Create the job:
   `hermes cron create "every monday 10am" "Run the docwatch skill on <repo path>. If every doc is still valid, reply with exactly [SILENT] and nothing else. If any doc drifted, send the full drift report." --deliver telegram:<chat id>`
3. Attach the skill: `hermes cron edit <job id> --name "Docwatch" --skill docwatch`
4. Put `hooks/post-commit` in your repo's `.git/hooks/`, replace the job id, and run `chmod +x` on it.
5. Optional: set `cron.wrap_response: false` in `~/.hermes/config.yaml` for clean messages.
