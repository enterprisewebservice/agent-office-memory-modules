---
name: claude-code
description: Use Claude Code CLI for complex reasoning, coding, shell commands, and multi-step investigations
metadata: { "openclaw": { "emoji": "🧠", "requires": { "bins": ["claude"] } } }
---

# Claude Code Skill — Powerful Reasoning & Coding via Claude Code CLI

You have access to **Claude Code**, a full-featured AI coding assistant running as a CLI tool. It can read/write files, run shell commands, search codebases, and perform multi-step investigations — all through your Max subscription (no per-request API cost).

## When to Use Claude Code

**Use it for:**
- Multi-step coding tasks (write code, run tests, fix errors, iterate)
- Investigating complex issues (read logs, search code, trace execution paths)
- Shell command sequences that need reasoning between steps
- Refactoring, debugging, or reviewing code
- Any task that benefits from persistent context across multiple interactions

**Don't use it for:**
- Simple questions you can answer from your own knowledge
- Single-command operations (just use `exec` directly)
- Quick file reads (use your own tools)
- Tasks where the user wants instant responses (Claude Code adds latency)

## Tools Reference

### `claude_code_resume` (Primary — use this by default)

```
claude_code_resume({
  prompt: "Your prompt here",
  taskLabel: "descriptive-label",
  agentId: "your-agent-id"
})
```

**Always provide a `taskLabel`** — it's how sessions are tracked and resumed.

### `claude_code_query` (Explicit control)

Start a new session or resume by specific session ID.

### `claude_code_sessions`

List your sessions with status, cost, and message count.

### `claude_code_fork`

Branch from an existing session to explore an alternative approach.

### `claude_code_kill`

Mark a session as done when finished.

## Session Decision Tree

1. **Continuing existing task?** → `claude_code_resume` with same `taskLabel`
2. **New task?** → `claude_code_resume` with new `taskLabel`
3. **Different approach?** → `claude_code_fork` from current session
4. **Task done?** → `claude_code_kill` to clean up

## How to Relay Results

- **Summarize** key findings — don't dump raw output
- **Quote specific code** if the user needs to see it
- **Report the sessionId** if the user might want to continue later
- **Mention cost** only if significant (> $0.10)
