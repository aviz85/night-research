# Learnings

## What Worked Well

### 1. Constraint-Driven Design
The discovery that subagents can't nest forced a better architecture. The 2-phase fan-out is actually cleaner than the original 3-tier pyramid — the main orchestrator has full visibility into all results and can do cross-project analysis that nested agents couldn't.

### 2. The claude-code-guide Agent
Using a specialized agent to research Claude Code's own capabilities before designing the system saved hours of trial-and-error. The nesting constraint would have been discovered painfully at runtime otherwise.

### 3. Structured Output Formats
The `=== SECTION ===` format for agent outputs is a sweet spot — human-readable but machine-parseable. JSON is too fragile (agents sometimes malform it), free-form text is too hard to parse.

### 4. Model Stratification
Using sonnet for discovery (needs reasoning) and haiku for questions (just reading files) cuts cost ~60% with minimal quality loss. The key insight: leaf agents doing mechanical work don't need expensive models.

### 5. Explore Subagent as Read-Only Enforcement
By using Explore agents (which lack Write/Edit tools), read-only is enforced at the platform level — not just via prompt instructions. Defense in depth.

## What Didn't Work (or Needed Pivoting)

### 1. Original 3-Tier Pyramid Design
The dream architecture (Main → Project Agents → Question Agents) was blocked by Claude Code's single-level nesting constraint. Lesson: always verify platform capabilities before designing around assumptions.

### 2. Batch Size Estimation
Initial estimates of "3-4 for Phase 1, 10 for Phase 2" are theoretical. Real-world performance depends on project size, agent response times, and system load. May need tuning.

### 3. Holding Results in Context Memory
The original design stored all discovery and answer results in the main session's context window. With ~110 agents producing output, the context filled up and was compacted — losing all intermediate results. We had to mine the JSONL transcript file with Python to recover data. Fix: save every batch's output to disk immediately using Write tool. Synthesis reads from files, not memory.

### 4. Bash `case` Statement in Claude Code
The project detection script used a bash `case` statement with pipe-separated patterns. This silently failed in Claude Code's Bash tool. Fix: replaced with `grep -qE` pattern matching which works reliably.

### 5. Silent Agent Output Loss
Some Task tool calls return only an `agentId` without the agent's text output. The work was done but the result is invisible to the orchestrator. Fix: check for empty outputs and re-run the agent immediately.

## Surprises and Discoveries

### Subagent Nesting Limitation
The biggest surprise. The Task tool's `general-purpose` type has "Tools: *" in the docs, which suggests full access including Task. But the claude-code-guide confirmed subagents cannot spawn subagents. This is an architectural constraint, not a tool-access limitation.

### Prompt Template as Architecture
The prompt templates (discovery-agent.md, question-agent.md) are essentially the "code" of this system. There's no traditional programming — the entire behavior is defined by natural language prompts with structured output formats.

### Context Limit Blowout (`classifyHandoffIfNeeded`)
When subagents exhaust their context window, Claude Code throws `classifyHandoffIfNeeded is not defined`. This is an internal error — not a user-configurable hook. The fix: always set `max_turns` on Task tool calls to cap agent runtime before they hit context limits. Discovery agents: 20 turns. Question agents: 12 turns.

## Recommendations for Similar Projects

1. **Research the platform first** — Use claude-code-guide or equivalent to understand constraints before designing
2. **Fan-out > nesting** — In Claude Code, a flat/2-phase fan-out is more reliable than trying to nest agents
3. **Structured output > JSON** — For agent-to-agent communication, use delimited sections, not JSON
4. **Model-per-role** — Don't use the same model for every agent; match model capability to task complexity
5. **Batch conservatively** — Start with small batches and scale up, not the other way around
6. **Read-only by design** — Use Explore agents for anything that shouldn't modify files; prompt instructions alone aren't enough
7. **Save to disk, not memory** — Never hold intermediate results only in context. Write files after every batch. Context can be compacted at any time
8. **Design for recovery** — Add a Step 0 that checks for existing partial results. The pipeline should be idempotent and resumable
9. **Avoid bash `case` in Claude Code** — Use `grep -qE` for pattern matching instead. The `case` statement silently fails
10. **Check for empty agent output** — Task tool sometimes returns only agentId. Detect and re-run immediately
