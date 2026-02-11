# Tools and APIs

## Core Platform

### Claude Code CLI
- **Role:** Main orchestrator + all agent execution
- **Key Feature:** Task tool for spawning subagents
- **Constraint:** Only 1 level of nesting (subagents can't spawn subagents)

### Task Tool — Subagent Types

| Type | Tools Available | Used For |
|------|----------------|----------|
| `Explore` | Read, Glob, Grep | Discovery agents + Question agents (read-only) |
| `general-purpose` | All tools incl. Task | Not used (nesting not needed in final design) |

### Model Selection

| Agent Role | Model | Why |
|-----------|-------|-----|
| Discovery (Phase 1) | sonnet (inherited) | Needs reasoning to generate good questions |
| Questions (Phase 2) | haiku | Fast, cheap, sufficient for file reading |
| Synthesis (Phase 3) | opus (main session) | Needs deep reasoning for cross-project analysis |

## File System Tools

### Glob
- Pattern-based file discovery
- Used by agents to find relevant source files
- Example: `Glob("src/**/*.ts", path="/Users/aviz/project")`

### Grep
- Content search across files
- Used by agents to find specific patterns (TODOs, secrets, imports)
- Example: `Grep("TODO|FIXME|HACK", path="/Users/aviz/project")`

### Read
- File content reading
- Used by agents to examine specific files
- Supports line offsets for large files

## Communication

### WhatsApp (via WAHA + Green API)
- **Skill:** `~/.claude/skills/whatsapp/`
- **Used for:** Final summary delivery
- **Endpoint:** Aviz's number (972503973736)

## Cost Estimates (per run)

| Component | Count | Model | Est. Tokens | Est. Cost |
|-----------|-------|-------|-------------|-----------|
| Discovery agents | 10 | sonnet | ~50K each | ~$0.75 |
| Question agents | 100 | haiku | ~10K each | ~$0.25 |
| Synthesis | 1 | opus | ~100K | ~$1.50 |
| **Total** | **111** | **mixed** | **~1.6M** | **~$2.50** |

## Gotchas and Limitations

1. **No nesting** — Subagents cannot spawn subagents. Main must orchestrate all levels.
2. **Context limits** — Each Explore agent has limited context. Questions must be focused.
3. **Batch sizing** — Too many parallel agents can overwhelm the system. 3-4 for Phase 1, 10 for Phase 2.
4. **Read-only enforcement** — Explore agents don't have Write/Edit, but READ-ONLY is also emphasized in prompts as defense-in-depth.
5. **Template parsing** — Discovery agent output must follow exact format for orchestrator to extract questions.
