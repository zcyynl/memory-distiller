---
name: memory-distiller
version: 1.1.0
description: "OpenClaw's subconscious. Automatically distills conversation insights, corrections, and preferences into durable memory. The agent that learns from every session — so you never have to repeat yourself."
author: zcyynl
---

# memory-distiller 🧠

> **The subconscious of your OpenClaw agent.** Automatically distills fleeting conversation moments into permanent wisdom.

Most agents wake up blank. memory-distiller changes that — it automatically identifies what's worth remembering during each conversation, writes it to persistent memory files, and creates a true learning loop. Your agent gets smarter every session.

---

## Why You Need It

OpenClaw natively has `MEMORY.md` and `memory/YYYY-MM-DD.md`, but updating them requires manual effort.

The problem: **You correct the agent, it says "got it", and makes the same mistake next session.**

memory-distiller closes this loop:
- Automatically scans conversations for "worth remembering" signals
- Applies a quality gate to filter out one-time, temporary information
- Writes structured entries to memory files automatically

---

## How It Fits With proactive-agent

| Skill | Role | Responsibility |
|-------|------|----------------|
| **proactive-agent** | Butler | Real-time detail capture (WAL), proactive behaviors, Heartbeat |
| **memory-distiller** | Historian | Post-conversation reflection, distilling lessons into long-term memory |

One line: **proactive-agent owns the present. memory-distiller owns the future.**

---

## Scheduled Daily Distillation (Cron Mode)

### Setup (One-Time)

After installing this skill, set up a daily cron job to auto-distill every night:

```javascript
// Run this in your OpenClaw session to set up the daily cron:
// Ask your agent: "Set up memory-distiller daily cron at 22:00"
```

The agent will create a cron job like this:

```json
{
  "name": "memory-distiller daily",
  "schedule": {
    "kind": "cron",
    "expr": "0 22 * * *",
    "tz": "Asia/Shanghai"
  },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "You are running the memory-distiller daily distillation job.\n\n1. Use sessions_list to find today's sessions\n2. Use sessions_history to read today's conversations\n3. Apply the memory-distiller quality gate to identify what's worth keeping\n4. Write distilled entries to memory/YYYY-MM-DD.md (today's date)\n5. Update MEMORY.md if any persistent rules or preferences were found\n6. Send a brief summary via Feishu DM to the user (message tool, channel: feishu)\n\nBe concise. If nothing is worth recording, still notify the user with '今日无新记忆写入'.",
    "timeoutSeconds": 300
  },
  "delivery": {
    "mode": "none"
  }
}
```

### When the User Says "Set up memory-distiller daily cron"

Immediately create the cron job using the `cron` tool:

```
cron(action="add", job={
  "name": "memory-distiller daily",
  "schedule": { "kind": "cron", "expr": "0 22 * * *", "tz": "Asia/Shanghai" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "You are running the memory-distiller daily distillation job.\n\n1. Use sessions_list to find today's sessions\n2. Use sessions_history to read today's conversations\n3. Apply the memory-distiller quality gate to identify what's worth keeping\n4. Write distilled entries to memory/YYYY-MM-DD.md (today's date)\n5. Update MEMORY.md if any persistent rules or preferences were found\n6. Send a brief Feishu DM summary to the user using message(action='send', channel='feishu', message='🧠 今日记忆整理完成\\n\\n[summary]')\n\nBe concise. If nothing is worth recording, notify the user with '今日无新记忆写入'.",
    "timeoutSeconds": 300
  },
  "delivery": { "mode": "none" }
})
```

Confirm to the user: "✅ 已设置每晚 22:00 自动记忆整理，完成后会飞书通知你。"

---

## Trigger Conditions

### Auto-Triggers (scan every user message for these signals)

| Type | Signal Words | Example |
|------|-------------|---------|
| 🔴 **Correction** | "wrong", "not right", "no, I meant", "actually", "stop doing" | "No, that command is wrong" |
| 💚 **Preference** | "I prefer", "always use", "don't use", "from now on", "by default" | "Always send reports as attachments" |
| 💡 **Insight** | "the issue was", "turns out", "the key is", "got it", "solved" | "Turns out Feishu doesn't render Markdown" |
| 📌 **Explicit** | "remember this", "save this", "note that", "write this down" | "Remember this config" |

### Manual Trigger

When the user says any of the following, immediately distill the session:

```
"remember this" / "save this" / "note that"
"write down what we just learned" / "distill this session"
```

---

## Quality Gate (All 4 Must Pass)

Before writing anything, check these 4 gates:

1. **Durability** — Will this still be valuable in 24 hours? (Skip one-time context)
2. **Generality** — Is this a reusable rule, or a one-off special case? (Prefer rules)
3. **Novelty** — Does `MEMORY.md` already contain this? (Avoid duplicates; update if stale)
4. **Actionability** — Can this guide future behavior? (Skip vague impressions; only concrete rules)

**Don't record (examples):**
- "Today we researched LangChain" → one-time, no guidance value
- "User is in Shanghai" → already in USER.md, duplicate

**Do record (examples):**
- "`clawhub install` only accepts slugs, not GitHub URLs" → actionable rule
- "Feishu chat does NOT render Markdown — long reports must be sent as attachments or doc links" → prevents repeated mistakes

---

## Memory Write Format

### Where to Write

| Content Type | Target File |
|-------------|-------------|
| Today's new discoveries, lessons | `memory/YYYY-MM-DD.md` |
| Important rules, persistent preferences | `MEMORY.md` (relevant section) |
| User personal info / preferences | `USER.md` |

### Entry Format

```markdown
### 🧠 Auto-Learned [YYYY-MM-DD HH:MM]
- **Type:** Correction / Preference / Insight / Explicit
- **Trigger:** One sentence explaining what triggered this
- **Rule:** Specific, actionable rule that can directly guide future behavior
```

**Example:**

```markdown
### 🧠 Auto-Learned [2026-03-02 00:30]
- **Type:** Correction
- **Trigger:** User corrected the install command format
- **Rule:** `clawhub install` only accepts slugs (e.g. claw-multi-agent), not GitHub URLs
```

---

## Execution Flow

```
User message arrives
    ↓
Scan for trigger signals (Correction / Preference / Insight / Explicit)
    ↓
Signal detected?
    ├─ No  → Reply normally, no action
    └─ Yes → Apply quality gate
                ↓
            All 4 gates pass?
                ├─ No  → Discard, reply normally
                └─ Yes → Distill into structured memory entry
                            ↓
                        Write to target file
                            ↓
                        Reply normally
                        (silent unless explicitly triggered)
```

---

## Behavioral Rules

### When to Notify the User

- **Auto-triggered** → Write silently, do NOT say "I've noted that" — don't interrupt the flow
- **Explicitly triggered** → Confirm with one line: "✅ Noted." + brief summary of what was recorded

### When NOT to Record

- User says "suppose...", "hypothetically...", "for example..." → hypothetical, skip
- System errors, network timeouts → environment issues, not learnable rules
- Content already fully documented in `MEMORY.md` → don't duplicate; update if stale

### Privacy Filter

**Never** write the following to any memory file, even if the user asks:
- Passwords, tokens, API keys
- Personal identification info (ID numbers, bank accounts, etc.)
- Sensitive information about third parties

---

## Installation

```bash
npx clawhub@latest install memory-distiller
```

After installation, restart your OpenClaw session, then say:

> **"Set up memory-distiller daily cron at 22:00"**

The agent will create a scheduled job that runs every night at 22:00 (Asia/Shanghai), distills the day's conversations, and sends you a Feishu DM summary.

---

## Recommended Pairing

| Skill | Purpose |
|-------|---------|
| **proactive-agent** | Real-time WAL protocol + proactive behaviors |
| **memory-distiller** | Post-conversation automatic experience distillation (this skill) |

Together: never lose present details, never repeat past mistakes.

---

## Design Philosophy

Inspired by [Claudeception](https://github.com/disler/claudeception) (a Claude Code self-learning plugin), redesigned from scratch for OpenClaw's architecture and memory model.

Core belief:
> **An agent should never be tripped up by the same problem twice.**

Every correction is a learning opportunity. memory-distiller ensures none of them go to waste.

Academic foundation: Voyager (2023), CASCADE (2024), SEAgent (2025), Reflexion (2023) — all point to the same conclusion: agents that persist and reflect on their learning dramatically outperform those that start fresh every time.

---

*Designed with 🐝 [claw-multi-agent](https://github.com/zcyynl/claw-multi-agent) — 3 parallel agents, 28s*
