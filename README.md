# Night Research — Pyramid Swarm

An autonomous multi-agent system built on [Claude Code](https://claude.ai/claude-code) that investigates your 10 most recent projects overnight. ~110 agents work in a 2-phase fan-out pyramid: first understanding each project, then answering 10 targeted research questions per project.

## How It Works

```
Main Orchestrator
├── Phase 1: Discovery (10 agents, batches of 3-4)
│   └── Explore each project → overview + 10 tailored questions
├── Phase 2: Investigation (100 agents, batches of 10)
│   └── Answer each question with file-level evidence
├── Phase 3: Synthesis
│   └── Per-project reports + cross-portfolio final report
└── Phase 4: Delivery
    └── WhatsApp summary with key findings
```

**Total: ~110 agents | ~1.6M tokens | ~$2.50 per run**

## Key Design Decisions

- **Read-only enforcement** — Explore agents lack Write/Edit tools. Prompts reinforce this as defense-in-depth
- **2-phase fan-out > nesting** — Claude Code subagents can't spawn subagents, so the main session orchestrates everything directly
- **Model stratification** — Sonnet for discovery (needs reasoning), Haiku for questions (just file reads), Opus for synthesis
- **Structured output** — `=== SECTION ===` delimiters instead of JSON. More robust for agent-to-agent communication
- **max_turns caps** — Prevents agents from hitting context limits and crashing

## Project Structure

```
night-research/
├── CLAUDE.md                      # Lean orchestration rules (loaded into context)
├── execution-guide.md             # Full pipeline steps (read at runtime)
├── prompts/
│   ├── discovery-agent.md         # Phase 1 prompt template
│   └── question-agent.md          # Phase 2 prompt template
├── templates/
│   ├── project-report.md          # Per-project output format
│   └── final-report.md            # Final synthesis format
├── docs/blog/                     # Design documentation (7 articles)
└── reports/                       # Generated output (gitignored)
```

## Context Window Optimization

CLAUDE.md is deliberately minimal (~40 lines). Detailed execution steps, templates, and report formats live in separate `.md` files that are only read when the pipeline runs. This keeps the context window lean for interactive use.

## Usage

Open Claude Code in this directory and say:

```
run the night research
```

The orchestrator reads `execution-guide.md` and runs the full pipeline autonomously.

## Requirements

- [Claude Code CLI](https://claude.ai/claude-code)
- WhatsApp skill configured (for delivery phase, optional)

## Documentation

The `docs/blog/` directory contains 7 articles documenting the design process:

1. **Project Overview** — Problem statement and goals
2. **Workflow Pipeline** — Architecture deep-dive
3. **Tools and APIs** — Platform capabilities and cost model
4. **Automation Created** — CLAUDE.md as executable architecture
5. **Results Comparison** — Why pyramid beats flat swarm
6. **Learnings** — What worked, what didn't, recommendations
7. **Agent Collaboration** — Meta-experience of building a swarm by talking to agents

## License

MIT
