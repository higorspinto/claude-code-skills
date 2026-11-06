---
name: context-session
description: Use when the user wants to start, resume, pause, save, checkpoint, list, or end a named work session. Trigger on: "start session", "resume session", "pause session", "save session", "checkpoint", "list sessions", "/session", "done for today", "I'll come back to this", "pick up where I left off", or any intent to persist, continue, or track work across Claude restarts or computer shutdowns. Always use this skill — never handle session commands ad-hoc.
---

# Context Session

Persistent named work sessions that survive shutdowns and restarts. Each session stores goal, plan, tasks, daily log, and notes as a Markdown file in `~/.claude/sessions/`.

## File Layout

```
~/.claude/sessions/
├── index.md          ← master list of all sessions
├── .active           ← single line: name of the current active session
├── auth-refactor.md
└── feature-x.md
```

Create `~/.claude/sessions/` if it doesn't exist.

## Session File Format

```markdown
---
name: auth-refactor
created: 2026-04-21
last_active: 2026-04-21
status: active
---

## Goal
What this session is trying to achieve.

## Plan
- [x] Completed step
- [ ] Current step
- [ ] Future step

## Completed Tasks
- Task name (2026-04-21)

## Pending Tasks
- Something not yet started

## Daily Log
### 2026-04-21
- What was done

## Notes
Decisions, gotchas, context that must survive a restart.
```

## Index Format (`index.md`)

```markdown
# Sessions

| Name | Status | Last Active | Goal |
|------|--------|-------------|------|
| auth-refactor | paused | 2026-04-21 | Refactor auth middleware |
```

---

## Commands

### Start a new session
**Triggers:** "start session X" / "/session start X"

1. If `<name>.md` already exists, ask: resume it or overwrite?
2. Ask the user for: goal, and a rough initial plan (can be one bullet point)
3. Create `~/.claude/sessions/<name>.md` using the format above
4. Write the session name to `.active`
5. Add a row to `index.md` (create the file if missing)
6. Confirm: *"Session '<name>' started. I'll keep it updated as we work."*

### Resume a session
**Triggers:** "resume session" / "resume session X" / "/session resume [X]"

- **Name given** → load `~/.claude/sessions/<name>.md` directly
- **No name given** → read `index.md`, show the 5 most recent sessions (newest `last_active` first) as a numbered list, ask the user to pick one

Once the session is chosen:
1. Read the session file fully — this is your working context
2. Write the name to `.active`
3. Set `status: active` and update `last_active` in the frontmatter
4. Add today's date header to Daily Log if not already present
5. Greet: *"Resuming '<name>'. Goal: [goal]. Last worked on: [most recent log entry or top pending task]."*

### List sessions
**Triggers:** "list sessions" / "/session list"

Read `index.md` and display the table. If the file doesn't exist, say *"No sessions yet. Start one with: start session <name>"*

### Checkpoint / save
**Triggers:** "checkpoint" / "save session" / "/session save" / "save"

1. Move any completed plan items to **Completed Tasks** (with today's date)
2. Sync **Pending Tasks** — remove done items, add newly identified ones
3. Append a log entry under today's date in **Daily Log** summarizing what happened
4. Update `last_active` in the frontmatter
5. Sync the row in `index.md`
6. Confirm: *"Saved. [one-line summary of what was logged]"*

### Pause session
**Triggers:** "pause session" / "done for today" / "/session pause"

1. Run a full checkpoint (same as above)
2. Set `status: paused` in the frontmatter
3. Clear `.active` (write an empty file or delete it)
4. Update `index.md`
5. Confirm: *"Session '<name>' paused. Resume anytime with: resume session <name>"*

### End / complete session
**Triggers:** "end session" / "/session end" / "session complete"

Same as pause, but set `status: completed`.

---

## Auto-Save During Active Sessions

When `.active` contains a session name, silently update the session file at these moments — no need to announce each save:

- A task from the plan is completed → move to Completed Tasks (with date)
- A new task is identified → add to Pending Tasks
- A significant decision is made → add a note to Notes
- Every 3–4 meaningful exchanges → append a quiet log entry

The goal is that if the user closes Claude mid-session without pausing, the session file is still useful when they come back. Keep entries brief and factual.

---

## Edge Cases

| Situation | Response |
|---|---|
| Resuming a name that doesn't exist | Say so, offer to start it instead |
| No sessions at all | "No sessions found. Start one with: start session \<name\>" |
| `.active` points to a deleted file | Treat as no active session; mention it |
| Session file is malformed | Tell the user and show the raw content |
