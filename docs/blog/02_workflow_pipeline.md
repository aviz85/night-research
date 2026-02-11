# The Workflow Pipeline

## Overview Architecture

```
                         ┌─────────────────────┐
                         │   Main Orchestrator  │
                         │   (Claude Code CLI)  │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │          Step 1: Detect        │
                    │   Find 10 most recent projects │
                    └───────────────┬───────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │            Phase 1: Discovery              │
              │    10 Explore agents (batches of 3-4)      │
              │    Each: understand project + 10 questions  │
              └─────────────────────┬─────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │           Phase 2: Investigation           │
              │   ~100 Explore agents (batches of 10)      │
              │   Each: answer 1 question with evidence    │
              └─────────────────────┬─────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │            Phase 3: Synthesis               │
              │   Main writes per-project + final report    │
              └─────────────────────┬─────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │            Phase 4: Delivery                │
              │         WhatsApp summary to Aviz            │
              └─────────────────────────────────────────────┘
```

## Step 1: Project Detection

### What We Did
Scan ~/  for directories that look like real projects (have .git, CLAUDE.md, or package.json). Filter out system directories. Sort by modification time. Take the top 10.

### Technical Details
```bash
for dir in $(ls -td /Users/aviz/*/); do
  name=$(basename "$dir")
  # Skip system dirs
  case "$name" in Desktop|Documents|Downloads|Library|...) continue;; esac
  # Check for project markers
  if [ -d "$dir/.git" ] || [ -f "$dir/CLAUDE.md" ] || [ -f "$dir/package.json" ]; then
    echo "$name"
  fi
done | head -10
```

### Why This Step
Not every directory is a project. We need project markers to avoid investigating Downloads or Library.

### Output
A list of 10 project names, ordered by recency.

## Step 2: Phase 1 — Discovery Agents

### What We Did
Spawn 10 Explore subagents via the Task tool, each assigned one project. They read CLAUDE.md, package.json, README, directory structure, and generate 10 research questions tailored to what they find.

### Technical Details
- Agent type: `Explore` (Read/Glob/Grep only — guaranteed read-only)
- Batching: 3-4 at a time to avoid overload
- Prompt template: `prompts/discovery-agent.md` with `{{PROJECT_PATH}}` and `{{PROJECT_NAME}}` substituted
- Output format: structured `=== OVERVIEW ===` + `=== QUESTIONS ===` blocks

### Why This Step
Generic questions waste agent capacity. By having discovery agents understand the project first, they generate questions that actually matter — "Why does the WhatsApp bot have no rate limiting?" instead of "Does the project have good practices?"

### Output
10 project overviews + 100 tailored questions.

## Step 3: Phase 2 — Question Agents

### What We Did
Spawn ~100 Explore subagents (haiku model for cost), each answering one question by investigating the actual source code.

### Technical Details
- Agent type: `Explore` with `model: haiku`
- Batching: 10 at a time
- Prompt template: `prompts/question-agent.md` with all variables substituted
- Output format: structured `=== Q# ===` blocks with answer, evidence, confidence

### Why This Step
One question per agent keeps context focused. Haiku is fast and cheap for file-reading tasks.

### Output
~100 structured answers with file evidence and confidence levels.

## Step 4: Synthesis

### What We Did
Main orchestrator groups all answers by project, writes per-project reports, then synthesizes the final report with cross-project insights.

### Output
- `reports/YYYY-MM-DD/projects/NAME.md` — 10 per-project reports
- `reports/YYYY-MM-DD/final-report.md` — portfolio-level analysis

## Step 5: Delivery

### What We Did
Send a concise WhatsApp summary with the top 3 insights and a pointer to the full report.
