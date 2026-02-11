# Execution Guide

Step-by-step pipeline. Read by orchestrator at runtime, not loaded into context by default.

## Step 1: Discover Projects

```bash
for dir in $(ls -td ~/*/); do
  name=$(basename "$dir")
  case "$name" in
    Desktop|Documents|Downloads|Library|Movies|Music|Pictures|Public|Applications|bin|sb|inbox|night-research|.Trash|node_modules|.npm|.nvm|.local|.config|.cache|.ssh|.claude|.cursor|.vscode|.zsh_sessions|.docker|.ollama) continue;;
  esac
  if [ -d "$dir/.git" ] || [ -f "$dir/CLAUDE.md" ] || [ -f "$dir/package.json" ]; then
    echo "$name"
  fi
done | head -10
```

A directory = project if it has `.git/` OR `CLAUDE.md` OR `package.json`.

## Step 2: Prepare Output

```bash
mkdir -p /Users/aviz/night-research/reports/$(date +%Y-%m-%d)/projects
```

## Step 3: Read Prompt Templates

- `prompts/discovery-agent.md`
- `prompts/question-agent.md`

## Step 4: Phase 1 — Discovery Agents

For each project → **Explore** subagent via Task tool.
- Substitute `{{PROJECT_PATH}}` and `{{PROJECT_NAME}}` in discovery-agent.md
- **Batch 3-4 at a time**, wait per batch
- **max_turns: 20** per agent (prevents context blowout)
- Parse returned text: extract overview + 10 questions

## Step 5: Phase 2 — Question Agents

For each ~100 questions → **Explore** subagent (model: **haiku**).
- Substitute all variables in question-agent.md
- **Batch 10 at a time**, wait per batch
- **max_turns: 12** per agent (focused reads, no sprawl)
- Each returns: structured answer with evidence

## Step 6: Synthesize Reports

Per-project: combine discovery overview + 10 answers → `reports/YYYY-MM-DD/projects/PROJECT_NAME.md`
Use template: `templates/project-report.md`

Final: synthesize all project reports → `reports/YYYY-MM-DD/final-report.md`
Use template: `templates/final-report.md`

## Step 7: WhatsApp Delivery

Send via whatsapp skill:

```
Night Research Complete

10 projects investigated, ~100 questions answered

Key findings:
- [Top insight 1]
- [Top insight 2]
- [Top insight 3]

Full report: ~/night-research/reports/YYYY-MM-DD/final-report.md
```
