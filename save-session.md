Snapshot the current session into `.swe-swe/TODO.md` so a future agent (or this one after a restart) can resume from a clean handoff.

## Steps

### 1. Check for an existing TODO.md

If `.swe-swe/TODO.md` already exists, show its contents to the user via `send_message` and ask whether to **overwrite**, **append**, or **abort**. Do not silently clobber a prior handoff.

### 2. Gather what to save

Reflect on the current conversation and assemble:

- **Goal** — one or two sentences: what is this session trying to accomplish?
- **Done** — concrete things already completed (files written, commits made, decisions reached). Reference paths and commit SHAs where useful.
- **Pending / Next steps** — an ordered checklist of what's left. Be specific enough that a fresh agent could pick up step 1 without re-deriving context.
- **Key files** — paths the next agent will need to read first.
- **Gotchas** — anything non-obvious: failed approaches, user preferences voiced this session, in-flight processes, dirty state, things to avoid.
- **How to verify** — how the next agent can confirm work is complete (tests, commands, expected output).

### 3. Write `.swe-swe/TODO.md`

Use the Write tool. Format as Markdown with the sections above as headings. Keep it tight and skimmable — this is a handoff note, not a journal.

### 4. Confirm

Report to the user via `send_message`: the path written, a one-line summary of what's pending, and remind them the file is gitignored (it stays local to this worktree).
