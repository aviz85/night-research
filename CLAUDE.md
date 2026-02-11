# Night Research — Pyramid Swarm

Autonomous multi-agent system. Investigates 10 most recent projects using ~110 agents in a 2-phase fan-out pyramid.

## Architecture

```
Main Orchestrator
├── Phase 1: Discovery (3-4 agents/batch, Explore, max_turns:20)
│   └── per project → overview + 10 questions
├── Phase 2: Investigation (10 agents/batch, Explore/haiku, max_turns:12)
│   └── per question → answer with evidence
├── Phase 3: Synthesis (main session)
│   └── per-project reports + final report
└── Phase 4: Delivery → WhatsApp summary
```

## Rules

1. **READ-ONLY** — No agent may create/modify/delete files in investigated projects
2. **Batching** — Phase 1: 3-4. Phase 2: 10. Wait per batch
3. **Output → reports/** — All generated files in `reports/YYYY-MM-DD/`
4. **No nesting** — Main orchestrates both phases directly
5. **max_turns** — Always set on Task calls (20 discovery, 12 questions) to prevent context blowout

## How to Run

Say "run the night research" → read `execution-guide.md` and follow all steps.

## Key Files

| File | Purpose |
|------|---------|
| `execution-guide.md` | Full pipeline steps (read at runtime) |
| `prompts/discovery-agent.md` | Phase 1 agent prompt template |
| `prompts/question-agent.md` | Phase 2 agent prompt template |
| `templates/project-report.md` | Per-project output format |
| `templates/final-report.md` | Final synthesis format |
