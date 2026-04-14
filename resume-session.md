Resume a session previously saved with `/ck:save-session`. Reads `.swe-swe/TODO.md`, confirms the plan with the user, then proceeds.

## Steps

### 1. Locate the handoff file

Check for `.swe-swe/TODO.md`. If it does not exist, tell the user "no saved session found" via `send_message` and stop.

### 2. Detect a stale resume

If `.swe-swe/TODO.resumed.md` **also** exists, a prior resume was started but never cleared. This is a warning sign — the previous agent may have begun the work and crashed, or may have completed it without housekeeping.

Show both files' contents to the user via `send_message` and ask:
- **Discard the old `TODO.resumed.md`** (assume it's stale, proceed with the fresh `TODO.md`), or
- **Use `TODO.resumed.md` instead** (continue what the prior agent started), or
- **Abort** and let the user clean up manually.

Do not proceed until the user picks one.

### 3. Read and restate the plan

Read `.swe-swe/TODO.md` (or `TODO.resumed.md` if the user chose that). Via `send_message`, restate back to the user:
- The goal
- What's already done
- The next concrete step you intend to take
- Any gotchas you noticed

Ask for confirmation to proceed. Do not act yet.

### 4. Claim the handoff

Once confirmed, **rename** `.swe-swe/TODO.md` → `.swe-swe/TODO.resumed.md` (use `mv` via Bash). This marks it as claimed so a second agent walking in won't act on it. If the user chose to continue from an existing `TODO.resumed.md` in step 2, skip the rename.

### 5. Proceed with the work

Execute the pending steps from the handoff. Use TaskCreate to track progress if the list is non-trivial.

### 6. On completion

Leave `.swe-swe/TODO.resumed.md` in place as a breadcrumb — it's gitignored, and it tells future agents "this handoff was picked up and worked on." The next `/ck:save-session` will create a new `TODO.md` alongside it.
