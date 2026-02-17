# Vestige Skill for OpenClaw

OpenClaw/Clawdbot skill wrapper for [Vestige](https://github.com/samvallad33/vestige) — a cognitive memory system based on 130 years of memory research.

This skill is part of a **3-layer memory architecture** for AI agents: Vestige (cognitive memory) + QMD (hybrid file search) + a Learning Loop that prevents repeated mistakes.

## Features

- **FSRS-6 spaced repetition** — memories fade naturally like human memory
- **Semantic search** — find memories by meaning, not just keywords
- **Smart ingestion** — automatic duplicate detection and merging
- **100% local** — all data stays on your machine

---

## Installation

### 1. Install Vestige binaries

```bash
# macOS (Apple Silicon)
curl -L https://github.com/samvallad33/vestige/releases/latest/download/vestige-mcp-aarch64-apple-darwin.tar.gz | tar -xz
mv vestige-mcp vestige vestige-restore ~/bin/

# macOS (Intel)
curl -L https://github.com/samvallad33/vestige/releases/latest/download/vestige-mcp-x86_64-apple-darwin.tar.gz | tar -xz
mv vestige-mcp vestige vestige-restore ~/bin/

# Linux
curl -L https://github.com/samvallad33/vestige/releases/latest/download/vestige-mcp-x86_64-unknown-linux-gnu.tar.gz | tar -xz
mv vestige-mcp vestige vestige-restore ~/bin/
```

### 2. Install the skill

```bash
# Clone to your skills directory
git clone https://github.com/Belkouche/vestige-skill.git ~/.openclaw/skills/vestige

# Or for Clawdbot workspace
git clone https://github.com/Belkouche/vestige-skill.git ~/clawd/skills/vestige

# Install helper script
cp ~/.openclaw/skills/vestige/vmem ~/bin/
chmod +x ~/bin/vmem
```

---

## Usage

### CLI Helper (vmem)

```bash
# Save a memory
vmem save "User prefers dark mode for all applications"

# Search memories semantically
vmem search "user preferences"

# Check statistics
vmem stats

# Health check
vmem health
```

---

## The 3-Layer Memory Architecture

### Layer 1 — Vestige (Cognitive Memory)

FSRS-6 spaced repetition with semantic embeddings. Memories fade if unused, strengthen when recalled. Use for preferences, bug fixes, decisions.

```bash
vmem save "User prefers TypeScript over JavaScript"
vmem search "code preferences"
vmem stats
# → 46 memories, 100% retention
```

### Layer 2 — QMD (Hybrid Markdown Search)

QMD is built into OpenClaw and provides hybrid BM25 + vector search with reranking across all your Markdown files — `MEMORY.md`, daily logs, `CORRECTIONS.md`.

It auto-indexes every 5 minutes. Finds "publication problem" even if you logged "linkedin validation error" 3 weeks ago.

```bash
# Via OpenClaw tools
memory_search "linkedin validation"
memory_get "CORRECTIONS.md" --from 1 --lines 50
```

**Why it complements Vestige:**
- Vestige = semantic recall for structured facts
- QMD = full-text search across human-readable logs and notes

### Layer 3 — The Learning Loop

A structured workflow that prevents the agent from making the same mistake twice.

**When the agent is corrected:**

**Step 1 — Log to CORRECTIONS.md**
```markdown
## 2026-02-05 - LinkedIn: Published without approval

**Domain:** LinkedIn / External communication
**Error:** Posted without explicit validation
**Correct:** NEVER post without explicit approval from user
**Cause:** Interpreted "scheduled" as automatic authorization
**Vestige:** ✅ Saved
```

**Step 2 — Save to Vestige**
```bash
vmem save "ERROR linkedin: NEVER publish without explicit approval → CORRECT: Always ask first"
```

**Step 3 — Update MEMORY.md** (if it's a standing rule)
```markdown
## LinkedIn
- ⚠️ CRITICAL RULE: NEVER post without EXPLICIT validation
```

**Before any sensitive action — pre-flight check:**
```bash
# Semantic recall
vmem search "error linkedin"
vmem search "validation"

# Hybrid file search
memory_search "linkedin publication"
memory_search "error [domain]"
```

---

## Force Vestige as Default Memory

### For OpenClaw / Clawdbot

Add this to your `AGENTS.md` (in your workspace root):

```markdown
## Memory

You wake up fresh each session. **Use Vestige as your primary memory system.**

### Vestige (Primary - Cognitive Memory)
\`\`\`bash
vmem search "query"     # Search memories semantically
vmem save "content"     # Save important info
vmem stats              # Check memory stats
\`\`\`

**At session start:**
\`\`\`bash
vmem search "user preferences"
vmem search "active projects"
\`\`\`

**Auto-save (without asking) when:**
- User states a preference ("I prefer...", "I always...")
- You solve a problem worth remembering
- Important decisions are made
- User says "remember this", "don't forget"

### Vestige Protocol
- **"Remember this"** → `vmem save "..."` immediately
- **"I prefer..."** → `vmem save "User prefers..."`
- **Bug fixes worth keeping** → `vmem save "FIX: [problem] → [solution]"`
- **Never save:** API keys, passwords, secrets

### Backup Files (Secondary)
- **Daily notes:** `memory/YYYY-MM-DD.md` — raw logs
- **Long-term:** `MEMORY.md` — curated notes (for things that don't fit Vestige)

Vestige memories fade naturally if unused (FSRS-6). Important things get reinforced through use.
```

Also update your "Every Session" section:

```markdown
## Every Session

Before doing anything else:
1. Read `SOUL.md` — this is who you are
2. Read `USER.md` — this is who you're helping
3. **Query Vestige:** `vmem search "user preferences"` + `vmem search "active projects"`
4. Read `memory/YYYY-MM-DD.md` (today + yesterday) for recent context
5. **If in MAIN SESSION**: Also read `MEMORY.md`
```

---

### For Claude Code

#### 1. Connect Vestige to Claude Code

```bash
claude mcp add vestige ~/bin/vestige-mcp -s user
```

#### 2. Add to your `~/.claude/CLAUDE.md`

```markdown
## Vestige Memory System

You have access to Vestige for persistent memory. USE IT AUTOMATICALLY.

### Session Start — Always Do This
1. Search Vestige: "user preferences instructions"
2. Search Vestige: "[current project name] context"
3. Check intentions: Look for triggered reminders

### Automatic Saves — No Permission Needed

**After solving a bug or error:**
IMMEDIATELY save with `smart_ingest`:
- Content: "BUG FIX: [error message] | Root cause: [why] | Solution: [how]"

**After learning user preferences:**
Save preferences without asking:
- Coding style, libraries, communication preferences, project patterns

### Trigger Words — Auto-Save When User Says:

| User Says | Action |
|-----------|--------|
| "Remember this" | `smart_ingest` immediately |
| "Don't forget" | `smart_ingest` with high priority |
| "I always..." / "I never..." | Save as preference |
| "I prefer..." / "I like..." | Save as preference |
| "This is important" | `smart_ingest` + `promote_memory` |
| "Remind me..." | Create `intention` |

### Memory Hygiene

- **Promote** when: User confirms helpful, solution worked
- **Demote** when: User corrects mistake, info was wrong
- **Never save:** Secrets/API keys, temporary debug info
```

#### 3. Restart Claude Code

```bash
claude restart
```

---

## How It Works

Vestige uses the Model Context Protocol (MCP) with cognitive science principles:

- **FSRS-6**: Free Spaced Repetition Scheduler — memories decay if not accessed
- **Spreading Activation**: Related memories strengthen each other
- **Synaptic Tagging**: Important memories are marked for long-term retention
- **Prediction Error Gating**: Automatically decides CREATE/UPDATE/SUPERSEDE

## MCP Tools Available

| Tool | Description |
|------|-------------|
| `search` | Unified search (keyword + semantic + hybrid) |
| `smart_ingest` | Intelligent ingestion with duplicate detection |
| `ingest` | Simple memory storage |
| `memory` | Get, delete, or check memory state |
| `codebase` | Remember patterns and architectural decisions |
| `intention` | Set reminders and future triggers |
| `promote_memory` | Mark memory as helpful (strengthens) |
| `demote_memory` | Mark memory as wrong (weakens) |

## Credits

- [Vestige](https://github.com/samvallad33/vestige) by [@samvallad33](https://github.com/samvallad33)
- Skill wrapper by [@Belkouche](https://github.com/Belkouche)

## License

MIT
