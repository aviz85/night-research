# Night Research - Pyramid Swarm

Autonomous multi-agent system that investigates the 10 most recent projects using ~110 agents in a 2-phase fan-out pyramid.

## Architecture

```
Main Orchestrator (this session)
|
+-- Phase 1: Discovery (batches of 3-4)
|   +-- Explore agent per project -> overview + 10 questions
|
+-- Phase 2: Investigation (batches of 10)
|   +-- Explore agent per question -> answer with evidence
|
+-- Phase 3: Synthesis (main)
|   +-- Per-project reports + final report
|
+-- Phase 4: Delivery
    +-- WhatsApp summary
```

## Rules

1. **READ-ONLY** - No agent may create, modify, or delete files in any investigated project
2. **Batching** - Phase 1: 3-4 agents at a time. Phase 2: 10 agents at a time
3. **Output only to reports/** - All generated files go in `reports/YYYY-MM-DD/`
4. **Subagents can't nest** - Main orchestrates both phases directly

## How to Run

Type `/run` or say "run the night research" to start the full pipeline.

## Project Detection

Find the 10 most recent project directories in ~/. A directory counts as a project if it has `.git/` OR `CLAUDE.md` OR `package.json`.

**Exclude these directories:**
Desktop, Documents, Downloads, Library, Movies, Music, Pictures, Public, Applications,
bin, sb, inbox, night-research, .Trash, node_modules, .npm, .nvm, .local, .config,
.cache, .ssh, .claude, .cursor, .vscode, .zsh_sessions, .docker, .ollama

**Detection command:**
```bash
for dir in $(ls -td /Users/aviz/*/); do
  name=$(basename "$dir")
  case "$name" in
    Desktop|Documents|Downloads|Library|Movies|Music|Pictures|Public|Applications|bin|sb|inbox|night-research|.Trash|node_modules|.npm|.nvm|.local|.config|.cache|.ssh|.claude|.cursor|.vscode|.zsh_sessions|.docker|.ollama) continue;;
  esac
  if [ -d "$dir/.git" ] || [ -f "$dir/CLAUDE.md" ] || [ -f "$dir/package.json" ]; then
    echo "$name"
  fi
done | head -10
```

## Execution Steps

### Step 1: Discover Projects
Run detection command. Store the 10 project names.

### Step 2: Prepare Output
```bash
mkdir -p /Users/aviz/night-research/reports/$(date +%Y-%m-%d)/projects
```

### Step 3: Read Prompt Templates
Read both files:
- `/Users/aviz/night-research/prompts/discovery-agent.md`
- `/Users/aviz/night-research/prompts/question-agent.md`

### Step 4: Phase 1 - Discovery Agents
For each project, spawn an **Explore** subagent via Task tool.
- Use the discovery-agent.md template with `{{PROJECT_PATH}}` and `{{PROJECT_NAME}}` substituted
- Batch 3-4 at a time. Wait for each batch before starting next
- Each agent returns: project overview + 10 numbered questions
- Parse the returned text to extract the 10 questions per project

### Step 5: Phase 2 - Question Agents
For each of the ~100 questions, spawn an **Explore** subagent (model: haiku).
- Use the question-agent.md template with all variables substituted
- Batch 10 at a time. Wait for each batch before starting next
- Each agent returns: structured answer with evidence

### Step 6: Synthesize Reports
For each project, combine discovery overview + 10 question answers into:
`reports/YYYY-MM-DD/projects/PROJECT_NAME.md`

Then synthesize all project reports into:
`reports/YYYY-MM-DD/final-report.md`

### Step 7: WhatsApp Delivery
Send concise summary using the whatsapp skill. Format:

```
Night Research Complete

10 projects investigated, ~100 questions answered

Key findings:
- [Top insight 1]
- [Top insight 2]
- [Top insight 3]

Full report: ~/night-research/reports/YYYY-MM-DD/final-report.md
```

## Report Formats

### Per-Project Report (projects/NAME.md)
```
# Project: NAME
Path | Tech Stack | Status | Maturity

## Overview
[From discovery agent]

## Research Findings
### Q1: [question]
[answer + evidence]
... (Q2-Q10)

## Top 3 Insights
## Recommended Actions
## Risk Assessment (tech debt / security / abandonment: Low/Med/High)
```

### Final Report (final-report.md)
```
# Night Research Report - DATE
Projects: 10 | Questions: ~100

## Executive Summary (2-3 paragraphs)
## Cross-Project Insights (themes, tech overview, maturity matrix)
## Individual Summaries (3-5 sentences each + key insight + recommendation)
## Prioritized Recommendations
## Portfolio Health Score
```
