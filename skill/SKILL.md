---
name: docwatch
description: Re-test docs against the live app; open PRs on drift.
version: 1.1.0
author: Himani Hassija
---

# Docwatch

Trigger: "docwatch <repo path>" (manual or scheduled).

## Steps
1. cd into the repo. Read docs/owners.yaml to know which docs to check and who owns each.
2. Run `git log -1 --format="%h %s (%an)"` and `git diff HEAD~1 --stat` to know the latest change.
3. Test like a brand-new teammate: run `pkill -f "uvicorn main:app"; sleep 1`, then `rm -rf /tmp/docwatch-run && git clone -q "$(git -C <repo path> remote get-url origin)" /tmp/docwatch-run && cd /tmp/docwatch-run`. Run ALL doc steps inside this fresh copy.
4. Run the Tech doc's steps in order. For its "start the server" step, run that exact command in the background (append `> server.log 2>&1 &`), then poll `curl -s -o /dev/null http://localhost:8000/` every 1s for up to 30s (any HTTP response means the server is up; do not depend on a specific endpoint). This counts as running that step. Never start the server a second time.
5. With the server running, run every step of the other docs EXACTLY as written, in order. Compare real output with the doc's "Expected". Record pass/fail and the real output.
6. For each failed step: read the current code and the latest git diff to find why. Write a corrected command and RUN it against the live app to prove it works.
7. Stop the server: `pkill -f "uvicorn main:app"`.
8. For each broken doc, open a pull request from /tmp/docwatch-run:
   - `git checkout -b docwatch/<short-hash>-<doc-name>`
   - Apply only the verified fixes to that doc.
   - `git -c user.name="Docwatch Bot" -c user.email="docwatch@leadly.dev" commit -am "docs: fix <doc> after <short-hash>"`
   - `git push -u origin <branch>`
   - `gh pr create --base main --head <branch> --title "docs: fix <doc> after <short-hash>" --body "<the drift report for this doc>"`
9. Report using the format below.

## Not doc drift
Failures caused by the test environment (port already in use, server not ready yet, leftover process, network blip) are NOT doc drift. Clean up and retry once. Only report drift if the step still fails and the cause is in the code or the doc.

## Report format
If everything passes:
Docwatch: all <N> docs are still valid at commit <hash>.

For each broken doc:
Doc drift detected
Doc: <path> (owner: <owner>)
Caused by: commit <hash> "<message>" by <author>
What broke: Step <N> expected <expected>, but returned <actual>.
Verified fix: <corrected command>
Pull request: <PR url>. The owner can merge it to accept the fix.

Use the same text as the PR body.

## Writing style
Write in plain, professional sentences everywhere: Telegram messages, PR titles, PR bodies and commit messages.
No emojis at all.
No em dashes or en dashes, and no bullet lists made with hyphens. Use separate lines with labels, or full sentences with commas and full stops instead.
File names, branch names and commands stay exactly as they are.

## Guardrails
- Never change application code. Only docs.
- Never push to main and never merge a PR. Merging is the owner's approval.
- Only run commands that appear in the docs, plus git/curl/pkill/gh for checking and PRs. No sudo.
