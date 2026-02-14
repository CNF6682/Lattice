# ORG Framework — Multi-Agent Organization for AI

A file-based organizational framework for managing multiple AI agents as a structured team. Built on [OpenClaw](https://github.com/openclaw/openclaw), but the principles apply to any multi-agent setup.

## What is this?

When you run multiple AI agents (coding assistants, researchers, reviewers, etc.), things get messy fast:
- No one knows what the other agent did last session
- Knowledge lives in chat history and dies with the session
- There's no quality gate — agents decide for themselves when "done" means done
- Reusable work gets rebuilt from scratch every time

ORG Framework solves this with:
- **File-based memory** — agents read/write Markdown files, not chat history
- **Department structure** — clear ownership, handoffs, and boundaries
- **Pipeline state machine** — 8-phase project lifecycle with quality gates
- **Reuse registry** — shared assets with ownership and access levels
- **Boot/Closeout rituals** — every session starts and ends the same way

## Quick Start

```
ORG/
├── ORG_README.md              # Start here — org entrypoint
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
│       │   ├── CONSTITUTION.md
│       │   ├── RESEARCH.md
│       │   ├── SPECIFICATION.md
│       │   ├── PLAN.md
│       │   ├── TASKS.md
│       │   ├── IMPL_STATUS.md
│       │   ├── TEST_REPORT.md
│       │   ├── REVIEW_REPORT.md
│       │   └── GAP_ANALYSIS.md
│       └── pipeline_archive/   # Historical run snapshots
└── ANNOUNCEMENTS/              # Cross-department announcements
```

## The Pipeline

The core of the framework. Every project with clear deliverables goes through 8 phases:

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

## Files Included

| File | Description |
|------|-------------|
| `ORG/ORG_README.md` | Organization entrypoint |
| `ORG/MEMORY_POLICY.md` | Memory layering rules |
| `ORG/PIPELINE_GUIDE.md` | Pipeline guide for all agents |
| `ORG/ASSET_REGISTRY.md` | Reusable asset registry (template) |
| `pipeline-framework/DESIGN.md` | Full pipeline design document |
| `pipeline-framework/templates/` | All prompt templates |
| `examples/` | Example department + project setup |

## Adapting to Your Setup

1. Copy the `ORG/` directory into your workspace
2. Create departments under `DEPARTMENTS/` using the charter template
3. Create projects under `PROJECTS/` — add `PIPELINE_STATE.json` for pipeline-driven ones
4. Set up an Orchestrator cron job (see `pipeline-framework/DESIGN.md` §5)
5. Configure agent roles and models in each project's `PIPELINE_STATE.json`

The framework is agent-runtime agnostic. It works with OpenClaw, but the file-based protocol means any agent that can read/write files can participate.

## License

MIT
