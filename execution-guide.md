# Execution Guide

Step-by-step pipeline. Read by orchestrator at runtime, not loaded into context by default.

## Step 0: Check for Previous Run (Recovery)

Before starting, check if a partial run exists:

```bash
ls reports/$(date +%Y-%m-%d)/discovery/ 2>/dev/null | wc -l
ls reports/$(date +%Y-%m-%d)/answers/ 2>/dev/null | wc -l
ls reports/$(date +%Y-%m-%d)/projects/ 2>/dev/null | wc -l
```

If files exist, **resume from where it stopped** — don't re-run completed phases:
- `discovery/PROJECT.md` exists → skip that project in Phase 1
- `answers/PROJECT.md` exists AND has 10 `=== Q` markers → skip that project in Phase 2
- `projects/PROJECT.md` exists → skip that project in Phase 3
- `final-report.md` exists → skip to Phase 4 (WhatsApp delivery)

Read `reports/YYYY-MM-DD/projects.txt` for the project list if it exists.

## Step 1: Discover Projects

Use a simple exclude-list grep approach (the `case` statement breaks in Claude Code's Bash tool):

```bash
EXCLUDE="Desktop|Documents|Downloads|Library|Movies|Music|Pictures|Public|Applications|bin|sb|inbox|night-research|\.Trash|node_modules|\.npm|\.nvm|\.local|\.config|\.cache|\.ssh|\.claude|\.cursor|\.vscode|\.zsh_sessions|\.docker|\.ollama"

ls -td ~/*/  | while read dir; do
  name=$(basename "$dir")
  echo "$name" | grep -qE "^($EXCLUDE)$" && continue
  if [ -d "$dir/.git" ] || [ -f "$dir/CLAUDE.md" ] || [ -f "$dir/package.json" ]; then
    echo "$name"
  fi
done | head -10
```

A directory = project if it has `.git/` OR `CLAUDE.md` OR `package.json`.

**SAVE the project list immediately:**
```bash
# Save so recovery can use it
echo "PROJECT_LIST" > reports/YYYY-MM-DD/projects.txt
```

## Step 2: Prepare Output

```bash
DATE=$(date +%Y-%m-%d)
mkdir -p /Users/aviz/night-research/reports/$DATE/{projects,discovery,answers}
```

Three subdirectories:
- `discovery/` — Phase 1 raw output (overview + questions per project)
- `answers/` — Phase 2 raw output (answers per project)
- `projects/` — Phase 3 synthesized reports

## Step 3: Read Prompt Templates

- `prompts/discovery-agent.md`
- `prompts/question-agent.md`

## Step 4: Phase 1 — Discovery Agents

For each project → **Explore** subagent via Task tool.
- Substitute `{{PROJECT_PATH}}` and `{{PROJECT_NAME}}` in discovery-agent.md
- **Batch 3-4 at a time**, wait per batch
- **max_turns: 20** per agent (prevents context blowout)
- Parse returned text: extract overview + 10 questions

### CRITICAL: Save after EVERY batch

After each batch returns, **immediately** write each agent's raw output to:
`reports/YYYY-MM-DD/discovery/PROJECT_NAME.md`

Use the Write tool — don't just hold it in memory. If context resets, files persist.

### Handling empty agent output

If an agent returns only an agentId with no text content, the agent's work is lost.
**Do not skip it** — re-run that single agent immediately with the same prompt.

## Step 5: Phase 2 — Question Agents

For each ~100 questions → **Explore** subagent (model: **haiku**).
- Substitute all variables in question-agent.md
- **Batch 10 at a time**, wait per batch
- **max_turns: 12** per agent (focused reads, no sprawl)
- Each returns: structured answer with evidence

### CRITICAL: Save after EVERY batch

After each batch of 10 returns, **immediately** write/append answers to:
`reports/YYYY-MM-DD/answers/PROJECT_NAME.md`

Each answer goes to the file matching its project. Use Write tool to save.

### Handling empty agent output

Same rule: if an agent returns only agentId without answer text, re-run it immediately.

## Step 6: Synthesize Reports

**Read from saved files** (NEVER from context memory):
- Discovery data: `reports/YYYY-MM-DD/discovery/PROJECT_NAME.md`
- Answer data: `reports/YYYY-MM-DD/answers/PROJECT_NAME.md`

Per-project: combine discovery + answers → `reports/YYYY-MM-DD/projects/PROJECT_NAME.md`
Use template: `templates/project-report.md`

Final: synthesize all project reports → `reports/YYYY-MM-DD/final-report.md`
Use template: `templates/final-report.md`

For synthesis, use **general-purpose** subagents (model: haiku) that read the files and write the reports.
Batch 5 at a time for per-project reports. Use sonnet for the final report.

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
