# Results Comparison

## Approach Comparison

| Approach | Agents | Depth | Cost | Time | Quality |
|----------|--------|-------|------|------|---------|
| Manual review (human) | 0 | Deep | Free | 5-10 hrs | High but inconsistent |
| Single agent (one session) | 1 | Shallow | ~$0.50 | 30 min | Low — context limit |
| Flat swarm (100 agents) | 100 | Medium | ~$2.50 | 20 min | Medium — no hierarchy |
| **2-phase pyramid** | **110** | **Deep** | **~$2.50** | **15-30 min** | **High — structured** |

## Why the 2-Phase Pyramid Wins

### vs Manual Review
- 10-20x faster
- Consistent methodology across all projects
- Never forgets to check something
- Runs while you sleep

### vs Single Agent
- Can't investigate 10 projects deeply in one context window
- No specialization — same agent does everything
- No parallelism

### vs Flat Swarm
- Discovery agents tailor questions to each project
- Questions are specific, not generic
- Results are structured for synthesis
- 2-phase = better signal-to-noise ratio

## Architecture Comparison

| Design | Nesting | Feasible? | Tradeoff |
|--------|---------|-----------|----------|
| True 3-tier pyramid | Main→Project→Question | No (Claude Code limitation) | Would be ideal but blocked |
| 2-phase fan-out | Main→Discovery, Main→Questions | Yes | Main does more work, but full control |
| Sequential chain | Main→Project1→Project2→... | Yes but slow | No parallelism |

## Model Selection Impact

| Scenario | Phase 1 Model | Phase 2 Model | Est. Cost | Quality |
|----------|--------------|---------------|-----------|---------|
| All opus | opus | opus | ~$15.00 | Overkill for file reading |
| All sonnet | sonnet | sonnet | ~$3.50 | Good but expensive for leaves |
| **Mixed (chosen)** | **sonnet** | **haiku** | **~$2.50** | **Best cost/quality ratio** |
| All haiku | haiku | haiku | ~$0.50 | Discovery questions too shallow |

## What To Watch For (Post-Run)

After the first run, evaluate:
1. **Question quality** — Are discovery agents generating specific, actionable questions?
2. **Answer depth** — Are haiku question agents providing enough evidence?
3. **Synthesis coherence** — Does the final report tell a useful story?
4. **False negatives** — Are there obvious issues the swarm missed?
5. **Batch stability** — Did any batches fail or timeout?
