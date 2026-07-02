---
description: Export the current chat log, commit only the exported files, then audit git history for content unsafe to push to a public repo
---

Do the following in order. Report progress with `send_progress` and the final result with `send_message`.

## 1. Export chat logs

Call the `mcp__swe-swe-agent-chat__export_chat_md` tool with a short kebab-case `title` describing this session (e.g. `auth-bug-fix`). This writes `./agent-chats/YYYY-MM-DD-NN-{title}.md`, copies any uploaded image attachments into `./agent-chats/assets/`, and upserts `./agent-chats/index.html`.

Capture the exact paths it touched (the `.md` file, `index.html`, and — on a first-ever export — `assets/viewer.css`, `assets/viewer.js`, plus any image assets).

## 2. Commit only the targeted files

Stage ONLY the files the export actually created or modified. **Never use `git add -A` or `git add .`** — pass explicit paths:

```
git add agent-chats/<the-new-file>.md agent-chats/index.html
# plus agent-chats/assets/... only if the export wrote assets this run
```

Run `git status --short agent-chats/` first to confirm exactly what changed, then stage those paths. Commit with a message like:

```
docs(agent-chats): export session log (<short topic>)
```

### Merge / cherry-pick conflicts on index.html

When this commit is later merged or cherry-picked onto a branch that already
has a same-day export, `index.html` may conflict on the manifest array, and two
entries can end up sharing the same date and `NN` index. That is fine: leave the
duplicate date + index alone. The `NN` and index are cosmetic labels, not keys.

Only a full `.md` filepath collision matters (two entries pointing at the exact
same `YYYY-MM-DD-NN-{title}.md` path). Different title slugs are NOT a collision,
even at the same date + `NN` -- do NOT rename or renumber to avoid duplicate
indexes. Resolve a manifest conflict by keeping BOTH entries (newest first) and
nothing else. Only if the exact filepaths would collide, bump `NN` on the new
file (and its manifest entry) to the next free number.

## 3. Audit history for public-repo safety

Delegate this step to the `/scrub-git-history` command, scoped to the export:

```
/scrub-git-history agent-chats/
```

That command audits the staged diff and recent `agent-chats/` history for secrets, PII, and infra leakage, and reports the result. Follow its verdict: if it flags anything, hold off (do not push) and surface its findings to the user; if it comes back clean, confirm the commit and the file(s) committed via `send_message`.

Do not push to any remote as part of this command — pushing is the user's decision.
