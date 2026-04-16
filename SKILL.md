---
name: tracekit
description: "Capture, list, and analyze coding-agent session traces for token and cost inefficiencies. Parses traces from Claude Code, OpenCode, and Codex to detect retry loops, redundant reads, context bloat, edit cascades, and tool fanout. Use when reviewing session costs, analyzing token usage, finding expensive sessions, detecting agent inefficiency patterns, generating cost reports, or capturing traces at session end."
---

# tracekit

Analyze coding-agent session traces to find token waste and cost inefficiencies across Claude Code, OpenCode, and Codex.

## Workflow

### 1. List sessions

```bash
tracekit list sessions
tracekit list sessions --agent claude
tracekit list sessions --since 2026-01-01
tracekit list sessions --format json
```

### 2. Analyze a session

```bash
tracekit analyze session --session-id <id>
tracekit analyze recent --limit 10
tracekit analyze expensive --top 20
tracekit analyze session --session-id <id> --format json
```

### 3. Generate a report

```bash
tracekit report session --session-id <id>
tracekit report session --session-id <id> --format html --out report.html
tracekit report aggregate --format html --out report.html
tracekit report session --session-id <id> --format json --out report.json
```

### 4. Capture traces

```bash
tracekit capture all
tracekit capture recent --limit 5
tracekit capture session --session-id <id>
```

## Common patterns

**Analyze current Claude Code session:**
```bash
tracekit analyze session --session-id <current-session-id> --agent claude
```

**Find most wasteful sessions:**
```bash
tracekit analyze expensive --top 20 --format json | jq '.sessions[] | {id: .session.session_id, cost: .session.total_cost_usd, findings: (.findings | length)}'
```

**Filter by inefficiency type:**
```bash
tracekit analyze recent --limit 20 --format json | jq '[.sessions[].findings[] | select(.kind == "retry_loop")]'
```

**Weekly cost report:**
```bash
tracekit report aggregate --since $(date -u -v-7d +%Y-%m-%d) --format html --out weekly-report.html
```

## Findings reference

| Finding | Fix |
|---|---|
| `RETRY_LOOP` | Fix the underlying tool error or add error-handling instructions. |
| `EDIT_CASCADE` | Check file permissions or patch format. |
| `TOOL_FANOUT` | Use batch tool calls or parallel execution. |
| `REDUNDANT_REREAD` | Cache file content in context instead of re-reading. |
| `CONTEXT_BLOAT` | Compress tool outputs or use summarization. |
| `ERROR_REPROMPT_CHURN` | Add explicit error-handling paths to system prompt. |
| `SUBAGENT_OVERHEAD` | Evaluate if tasks need subagents or can be done inline. |

## Notes

- Claude Code and OpenCode provide real cost data; Codex supports structural analysis only
- Session IDs support prefix matching (first 8 chars usually sufficient)
- Use `--agent all` (default) to search across all installed agents
- Binary location: `./target/release/tracekit` or `tracekit` if installed globally
