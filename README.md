# claude-code-skills

Custom skills for [Claude Code](https://claude.ai/code).

## Skills

### `context-session`

Persistent named work sessions that survive Claude restarts and computer shutdowns. Session state (goal, plan, tasks, daily log, notes) is stored as Markdown files in `~/.claude/sessions/`.

**Triggers automatically on:** "start session", "resume session", "pause session", "checkpoint", "list sessions", "done for today", "pick up where I left off"

**Commands:**

| Command | Example |
|---------|---------|
| Start a session | `start session auth-refactor` |
| Resume a session | `resume session` or `resume session auth-refactor` |
| List all sessions | `list sessions` |
| Save progress | `checkpoint` or `save session` |
| Pause for later | `pause session` or `done for today` |
| Mark complete | `end session` |

Sessions are stored in `~/.claude/sessions/` as Markdown files with frontmatter tracking name, status, and last-active date.

## Installation

Copy the skill folder(s) into your Claude Code skills directory:

```bash
cp -r context-session ~/.claude/skills/
```

Skills are picked up automatically by Claude Code on the next session.
