# Vestige Skill for OpenClaw

OpenClaw/Clawdbot skill wrapper for [Vestige](https://github.com/samvallad33/vestige) - a cognitive memory system based on 130 years of memory research.

## Features

- **FSRS-6 spaced repetition** - memories fade naturally like human memory
- **Semantic search** - find memories by meaning, not just keywords
- **Smart ingestion** - automatic duplicate detection and merging
- **100% local** - all data stays on your machine

## Installation

### 1. Install Vestige binaries

```bash
# macOS (Apple Silicon)
curl -L https://github.com/samvallad33/vestige/releases/latest/download/vestige-mcp-aarch64-apple-darwin.tar.gz | tar -xz
mv vestige-mcp vestige vestige-restore ~/bin/

# macOS (Intel)
curl -L https://github.com/samvallad33/vestige/releases/latest/download/vestige-mcp-x86_64-apple-darwin.tar.gz | tar -xz
mv vestige-mcp vestige vestige-restore ~/bin/
```

### 2. Install the skill

```bash
# Clone to your skills directory
git clone https://github.com/Belkouche/vestige-skill.git ~/.openclaw/skills/vestige

# Or for Clawdbot
git clone https://github.com/Belkouche/vestige-skill.git ~/clawd/skills/vestige

# Install helper script
cp ~/.openclaw/skills/vestige/vmem ~/bin/
chmod +x ~/bin/vmem
```

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

### In OpenClaw/Clawdbot

The skill is automatically loaded. Use trigger phrases:

| Say | Action |
|-----|--------|
| "Remember this..." | Saves to Vestige |
| "I prefer..." | Saves as preference |
| "What do you know about..." | Searches Vestige |

## How It Works

Vestige uses the Model Context Protocol (MCP) with cognitive science principles:

- **FSRS-6**: Free Spaced Repetition Scheduler - memories decay if not accessed
- **Spreading Activation**: Related memories strengthen each other
- **Synaptic Tagging**: Important memories are marked for long-term retention

## Credits

- [Vestige](https://github.com/samvallad33/vestige) by [@samvallad33](https://github.com/samvallad33)
- Skill wrapper by [@Belkouche](https://github.com/Belkouche)

## License

MIT
