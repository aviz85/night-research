# Automation Created

## System Architecture

Night Research is itself an automation — a self-contained system that runs autonomously and produces actionable intelligence.

### CLAUDE.md Structure
The project's CLAUDE.md serves as the orchestration brain:
- Project detection logic (bash command)
- 7-step execution flow
- Batching rules
- Report format specifications
- WhatsApp delivery format

### Prompt Templates

Two template files drive the agent behavior:

#### `prompts/discovery-agent.md`
- Variables: `{{PROJECT_PATH}}`, `{{PROJECT_NAME}}`
- Instructs Explore agents to: read project files, understand structure, generate 10 tailored questions
- Enforces structured output format (`=== OVERVIEW ===` + `=== QUESTIONS ===`)
- Includes specific Glob patterns for common project structures

#### `prompts/question-agent.md`
- Variables: `{{PROJECT_PATH}}`, `{{PROJECT_NAME}}`, `{{QUESTION_NUMBER}}`, `{{QUESTION_TEXT}}`
- Instructs Explore agents to: investigate one question, cite evidence, rate confidence
- Enforces structured output format (`=== Q# ===`)
- Limits answers to 100-300 words for efficient synthesis

## How Claude Code Agent Helped

### Architectural Constraint Discovery
The original design was a 3-tier pyramid where project agents would spawn question agents. During planning, the claude-code-guide agent revealed that subagents cannot spawn subagents. This led to a cleaner 2-phase fan-out design where the main orchestrator handles both levels.

### Iterative Design
The architecture went through 3 iterations:
1. **v1:** True pyramid (main → project agents → question agents) — blocked by nesting constraint
2. **v2:** Flat fan-out (main → all 110 agents) — too unstructured
3. **v3:** 2-phase fan-out (main → 10 discovery → 100 questions) — preserves hierarchy without nesting

### Prompt Engineering
Discovery agent prompts are designed to produce machine-parseable output while still allowing natural language reasoning. The `=== OVERVIEW ===` / `=== QUESTIONS ===` format is both human-readable and easily extractable.

## Reusable Components

### The 2-Phase Fan-Out Pattern
Any workflow that needs hierarchical delegation in Claude Code can use this pattern:
1. Phase 1: Explore agents gather context and generate subtasks
2. Phase 2: More Explore agents execute each subtask
3. Main synthesizes all results

### Structured Agent Output Templates
The `=== SECTION ===` format is a reliable way to get parseable output from subagents without requiring JSON (which agents sometimes malform).

### Project Detection Logic
The bash one-liner for finding real projects (vs system directories) is reusable across any project-portfolio tool.
