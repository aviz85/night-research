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

## Disclaimer

This is an **experiment** — a proof-of-concept for multi-agent orchestration patterns with Claude Code. It is not production software. Use at your own risk. The authors take no responsibility for API costs, unexpected behavior, or any consequences of running this system. You are solely responsible for reviewing the code and understanding what it does before executing it.

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
    └── YYYY-MM-DD/
        ├── projects.txt           # Discovered project list (checkpoint)
        ├── discovery/             # Phase 1 raw agent output (checkpoint)
        ├── answers/               # Phase 2 raw agent output (checkpoint)
        ├── projects/              # Phase 3 per-project reports
        └── final-report.md        # Final synthesis
```

## Context Window Optimization

CLAUDE.md is deliberately minimal (~40 lines). Detailed execution steps, templates, and report formats live in separate `.md` files that are only read when the pipeline runs. This keeps the context window lean for interactive use.

## Crash Recovery

The pipeline is designed to survive context window exhaustion (the most common failure mode). Every phase saves results to disk immediately after each batch completes:

```
reports/YYYY-MM-DD/
├── projects.txt          # Project list (saved in Step 1)
├── discovery/            # Phase 1 raw output (saved per batch)
├── answers/              # Phase 2 raw output (saved per batch)
├── projects/             # Phase 3 synthesized reports
└── final-report.md       # Final synthesis
```

If the context runs out mid-pipeline, just say "run the night research" again — the orchestrator checks what files already exist and resumes from where it stopped. No work is lost.

## Usage

Open Claude Code in this directory and say:

```
run the night research
```

The orchestrator reads `execution-guide.md` and runs the full pipeline autonomously.

## Requirements

- [Claude Code CLI](https://claude.ai/claude-code)
- WhatsApp skill configured (for delivery phase, optional)

## Known Issue: `classifyHandoffIfNeeded`

If subagents run too long and exhaust their context window, Claude Code throws an internal error: `classifyHandoffIfNeeded is not defined`. This is **not** a hook you configure — it's a Claude Code internal function that fails when an agent hits its context limit.

**This happens on its own** when agents do too many tool calls without a `max_turns` cap. The fix is already built into the execution guide: discovery agents are capped at 20 turns, question agents at 12. If you still see this error, lower the `max_turns` values or simplify the prompt templates to reduce agent work.

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
