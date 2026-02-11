# The Meta Experience: Building a Swarm by Talking to an Agent

## What Made This Different

This project is pure meta — an AI agent designing a system of AI agents. No code was written in the traditional sense. The entire system is natural language: CLAUDE.md orchestration rules, prompt templates with variable substitution, and structured output formats. The "programming language" is English with `=== SECTION ===` delimiters.

## Real Examples from This Project

### The Nesting Discovery
The original vision was a clean 3-tier pyramid: main agent spawns project agents, each project agent spawns 10 question agents. During planning, a claude-code-guide subagent was dispatched to research how the Task tool actually works. It came back with the critical finding: **subagents cannot spawn subagents**.

This forced an architectural pivot in real-time. Instead of abandoning the pyramid concept, the design was flattened into a 2-phase fan-out — preserving the hierarchy's intelligence (discovery before investigation) while respecting the platform's constraints. The resulting design is arguably better: the main orchestrator has full visibility into all 100+ answers and can do cross-project synthesis that nested agents couldn't.

### The Parallel Exploration
Three agents were dispatched simultaneously during planning:
1. One to explore the night-research directory and user's project landscape
2. One to find existing multi-agent patterns in Aviz's skills (review-swarm, fleet-orchestrator)
3. One to research the technical mechanics of agent nesting

All three returned within 40 seconds. The combined intelligence from these parallel explorations would have taken a human 30+ minutes of reading documentation and browsing files.

### Prompt Template Design
The prompt templates are the heart of the system. They had to be:
- Clear enough for agents to follow autonomously
- Structured enough for the orchestrator to parse results
- Flexible enough to work across wildly different projects (WhatsApp bots, CRM dashboards, video tools)

The solution: structured output blocks (`=== OVERVIEW ===`, `=== QUESTIONS ===`, `=== Q# ===`) that are both human-readable and machine-extractable.

## Skills Created During the Conversation

### The 2-Phase Fan-Out Pattern
A reusable architectural pattern for any Claude Code workflow that needs hierarchical delegation:
1. Phase 1: Explore agents gather context and generate subtasks
2. Phase 2: More agents execute each subtask (haiku for cost)
3. Main synthesizes

### Project Detection Logic
A reusable bash pattern for finding real projects vs system directories in a user's home folder.

### Structured Agent Communication
The `=== SECTION ===` format as a reliable alternative to JSON for agent-to-agent data passing.

## The Human's Role

Aviz's role was:
1. **Vision** — "I want swarms of agents investigating my projects"
2. **Architecture feedback** — Approving the 2-phase design after the nesting constraint was discovered
3. **Constraints** — "They must not touch the files, read-only!"
4. **Trigger** — Saying "run" when ready

Everything else — the research, the architecture, the prompt engineering, the file creation, the git setup — was handled by the agent.

## What This Demonstrates

### Natural Language as Infrastructure
The entire system has zero lines of traditional code. CLAUDE.md + prompt templates + the Task tool = a complete multi-agent orchestration platform. The "runtime" is Claude Code itself.

### Constraints Breed Better Design
The subagent nesting limitation could have been a blocker. Instead, it led to a cleaner architecture with better observability. Constraints don't kill ideas — they refine them.

### The Overnight Intelligence Loop
Night Research represents a new category of personal tool: autonomous intelligence that works while you sleep. Wake up to a report on your entire project portfolio. The Abraham Levy Principle (don't abandon working systems) enforced by automated oversight.
