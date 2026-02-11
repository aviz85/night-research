# Night Research: Autonomous Project Intelligence Through Agent Swarms

## The Challenge: "I have 30+ projects — what's actually going on in them?"

### Background Story
Aviz is a prolific builder. Claude Code skills, WhatsApp bots, CRM dashboards, video processing pipelines, insurance automation — dozens of projects accumulating in his home directory. Some are thriving, some are abandoned mid-thought, some have security issues nobody noticed. The problem isn't building — it's knowing what you've built.

### The Problem
No single person (or agent) can deeply investigate 10+ projects simultaneously. Reading every file, understanding every architecture, spotting every issue — it would take hours of manual review. And by the time you finish, you've already started 3 new projects.

### Why This Matters
Aviz has a documented pattern (the "Abraham Levy Principle"): the tendency to abandon working systems when excited about new things. Night Research is the antidote — an autonomous system that investigates your portfolio while you sleep and tells you what needs attention.

### Project Goals
1. Investigate the 10 most recently modified projects in ~/
2. Generate ~100 targeted research questions across all projects
3. Answer each question with evidence from actual source code
4. Synthesize per-project reports + a portfolio-level analysis
5. Deliver a WhatsApp summary so Aviz wakes up informed

### Why This Project is Interesting
It's a **meta-project** — an AI system that analyzes AI-built projects. It pushes Claude Code's agent orchestration to its limits: ~110 subagents, 2-phase fan-out, structured prompt templates with variable substitution, and hierarchical result synthesis. It's also a case study in working around platform constraints (subagents can't nest) with elegant architectural redesign.
