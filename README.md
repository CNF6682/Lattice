# Lattice — Multi-Agent Organization Framework

A file-based organizational framework for managing multiple AI agents as a structured team. Built on [OpenClaw](https://github.com/openclaw/openclaw), but the principles apply to any multi-agent setup.

## What is this?

When you run multiple AI agents (coding assistants, researchers, reviewers, etc.), things get messy fast:
- No one knows what the other agent did last session
- Knowledge lives in chat history and dies with the session
- There's no quality gate — agents decide for themselves when "done" means done
- Reusable work gets rebuilt from scratch every time

Lattice solves this with:
- **File-based memory** — agents read/write Markdown files, not chat history
- **Department structure** — clear ownership, handoffs, and boundaries
- **Pipeline state machine** — 8-phase project lifecycle with quality gates
- **Reuse registry** — shared assets with ownership and access levels
- **Boot/Closeout rituals** — every session starts and ends the same way

## Quick Start

### 1. Get the framework

```bash
git clone https://github.com/<owner>/lattice.git /tmp/lattice
cp -r /tmp/lattice/ORG <your-workspace>/ORG
```

### 2. Create your first department

```bash
cp -r ORG/DEPARTMENTS/example-dept ORG/DEPARTMENTS/my-team
```

Edit `CHARTER.md` with your team's mission, `HANDOFF.md` with current state.

### 3. Create your first project

```bash
cp -r ORG/PROJECTS/example-project ORG/PROJECTS/my-project
```

Edit `PIPELINE_STATE.json` — replace all `<your-xxx>` placeholders with your actual agent IDs and models.

### 4. Set up the Pipeline Orchestrator

Create a cron job that triggers every 30 minutes (see [DESIGN.md §5](ORG/PROJECTS/pipeline-framework/DESIGN.md) for the full template):

```json
{
  "name": "pipeline:my-project:orchestrator",
  "schedule": { "kind": "cron", "expr": "*/30 * * * *" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "You are the Pipeline Orchestrator. Read PIPELINE_STATE.json, advance the current phase by spawning sub-agents. Follow ORG/PROJECTS/pipeline-framework/templates/ORCHESTRATOR_PROMPT.template.md exactly.",
    "model": "<your-cheap-model>"
  }
}
```

### 5. Wire Boot Sequence into your agents

Add to `AGENTS.md` or system prompt:

```
Before any work: read ORG/TASKBOARD.md → your dept HANDOFF.md → ASSET_REGISTRY.md
After work: update HANDOFF.md and project STATUS.md
```

### 6. Watch it run

The orchestrator picks up the project, starts Phase 0 (Constitute), and drives through all 8 phases automatically. Each phase spawns a clean sub-agent session, produces a file artifact, and advances.

---

## The Pipeline

Every project with clear deliverables goes through 8 phases:

```
Phase 0: Constitute  → Define principles, constraints, quality bar
Phase 1: Research    → Survey existing work, identify risks
Phase 2: Specify     → Precise requirements + acceptance criteria
Phase 3: Plan+Tasks  → Break down into atomic tasks
Phase 4: Implement   → Code it (one task per clean session)
Phase 5: Test        → 3-level testing (unit → integration → acceptance)
Phase 6: Review      → Quality gate (PASS → Phase 7, FAIL → rollback)
Phase 7: Gap Analysis→ What's missing? What to improve next run?
```

Each phase:
- Runs in an isolated session (clean context)
- Has explicit entry/exit conditions
- Produces a file artifact (not chat messages)
- Is orchestrated by a lightweight scheduler agent

When Review passes, Gap Analysis runs, then everything archives to `pipeline_archive/run-{N}/` and the next run begins — with Gap Analysis feeding into the new Constitution.

## Key Concepts

### Memory Layering
- **Org level** — cross-project facts, policies, asset registry
- **Department level** — team state, handoffs, runbooks
- **Project level** — status, decisions, pipeline artifacts
- **Agent level** — personal workspace, drafts, preferences

### Boot / Closeout
Every agent, every session:
1. **Boot**: Read TASKBOARD → Department HANDOFF → ASSET_REGISTRY
2. **Closeout**: Update HANDOFF (done/next/blockers) → Update project STATUS → Register reusable assets

### Reuse Levels
- **L0** — Ready to use (has runbook + tests + stable I/O)
- **L1** — Usable with coordination (needs access/env setup)
- **L2** — One-off delivery (no reuse guarantee)

### Dual-Layer Assistance (when agents get stuck)
1. **Model Escalation** — automatically retry with stronger models along a cost chain
2. **Peer Consult** — parallel consultation with multiple models, then synthesize the best solution

## Deploy Guides

Two guides depending on how you want to do it:

| Guide | For | What it does |
|-------|-----|-------------|
| [Human Guide](docs/DEPLOY_GUIDE_HUMAN.md) | You (the human) | Copy-paste a prompt to your agent, it deploys everything |
| [Agent Runbook](docs/DEPLOY_GUIDE_AGENT.md) | Your agent | 10-phase step-by-step self-deployment runbook |

## Directory Structure

```
ORG/
├── ORG_README.md              # Org entrypoint
├── TASKBOARD.md               # Current priorities
├── MEMORY_POLICY.md           # What goes where (memory layering)
├── PIPELINE_GUIDE.md          # Pipeline framework guide (all agents)
├── ASSET_REGISTRY.md          # Reusable assets directory
├── REUSE_REQUESTS.md          # Cross-department reuse requests
├── DEPARTMENTS/
│   └── <dept>/
│       ├── CHARTER.md         # Mission, scope, boundaries
│       ├── RUNBOOK.md         # How to operate
│       ├── HANDOFF.md         # Current state / next / blockers
│       └── STATE.json         # Machine-readable state
├── PROJECTS/
│   └── <project>/
│       ├── STATUS.md          # Human-readable project status
│       ├── DECISIONS.md       # Key decisions with rationale
│       ├── PIPELINE_STATE.json # Pipeline state machine
│       ├── PIPELINE_LOG.jsonl  # Append-only phase transition log
│       ├── pipeline/           # Current run artifacts
│       └── pipeline_archive/   # Historical run snapshots
└── ANNOUNCEMENTS/              # Cross-department announcements
```

## Files Reference

| File | Description |
|------|-------------|
| `ORG/ORG_README.md` | Organization entrypoint |
| `ORG/MEMORY_POLICY.md` | Memory layering rules |
| `ORG/PIPELINE_GUIDE.md` | Pipeline guide for all agents |
| `ORG/ASSET_REGISTRY.md` | Reusable asset registry (template) |
| `ORG/PROJECTS/pipeline-framework/DESIGN.md` | Full pipeline design document |
| `ORG/PROJECTS/pipeline-framework/templates/` | All prompt templates (8 phases + orchestrator + consult) |
| `docs/DEPLOY_GUIDE_HUMAN.md` | Human deployment guide |
| `docs/DEPLOY_GUIDE_AGENT.md` | Agent deployment runbook |

## License

MIT
