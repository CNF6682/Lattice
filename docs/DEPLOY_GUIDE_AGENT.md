# 🤖 ORG Framework — Agent Deployment Runbook

> You are an AI agent on OpenClaw. Your human wants you to deploy the ORG Framework — a file-based multi-agent organization system with an 8-phase Pipeline for project management.
>
> Follow this runbook step by step. Do not skip steps. Ask your human when you need decisions (marked with 🧑).

---

## Phase 1: Acquire the Framework

```bash
git clone https://github.com/<owner>/org-framework.git /tmp/org-framework
```

Then copy the ORG directory into your workspace:

```bash
cp -r /tmp/org-framework/ORG <your-workspace>/ORG
```

Verify:
```bash
ls <your-workspace>/ORG/
# Expected: ASSET_REGISTRY.md  DEPARTMENTS/  MEMORY_POLICY.md  ORG_README.md  PIPELINE_GUIDE.md  PROJECTS/  REUSE_REQUESTS.md  TASKBOARD.md  ANNOUNCEMENTS/
```

---

## Phase 2: Understand the Framework

Read these files in order (do not skip):

1. `ORG/ORG_README.md` — entrypoint, boot sequence, hard rules
2. `ORG/MEMORY_POLICY.md` — memory layering (org / dept / project / agent)
3. `ORG/PIPELINE_GUIDE.md` — 8-phase pipeline overview
4. `ORG/PROJECTS/pipeline-framework/DESIGN.md` — full design doc (read §1-§4 minimum)

After reading, you should understand:
- Boot Sequence: TASKBOARD → HANDOFF → ASSET_REGISTRY
- Closeout Sequence: Update HANDOFF → STATUS → register reusable assets
- Pipeline phases: Constitute → Research → Specify → Plan → Implement → Test → Review → Gap Analysis
- Orchestrator role: schedule phases, never do the work itself

---

## Phase 3: Gather Information from Human

🧑 Ask your human the following (skip any they've already told you):

### 3.1 Organization
- What departments do you want? (e.g., "research", "engineering", "infrastructure")
- Brief mission for each department (1-2 sentences)

### 3.2 First Project
- Project name (e.g., "my-cool-project")
- One-line description
- Where is the code repo? (path)

### 3.3 Agent Roster
- What agents are available? List their agent IDs.
  - Run `agents_list` if unsure what's available.
- Which agent should play which Pipeline role?

| Role | Agent ID | Model |
|------|----------|-------|
| Architect (Phase 0, 3) | ? | ? |
| Researcher (Phase 1, 7) | ? | ? |
| Designer (Phase 2) | ? | ? |
| Coder (Phase 4, 5) | ? | ? |
| Reviewer (Phase 6) | ? | ? |

If only one agent exists, use it for all roles — just vary the model per phase.

### 3.4 Models
- What models are available? Check your OpenClaw config.
- Suggested allocation:
  - Strong model (for constitute/specify/plan/review): e.g., claude-opus, gpt-4o
  - Research model (for research/gap-analysis): e.g., gemini-pro, gpt-4o
  - Coding model (for implement/test): e.g., codex, claude-sonnet, deepseek-coder
  - Cheap model (for orchestrator): e.g., gemini-flash, gpt-4o-mini

### 3.5 Notification (optional)
- Where should Pipeline progress be announced? (Telegram topic, Discord channel, or none)

---

## Phase 4: Create Departments

For each department the human specified:

```bash
# Copy template
cp -r ORG/DEPARTMENTS/example-dept ORG/DEPARTMENTS/<dept-name>
```

Edit each file:
- `CHARTER.md` — Fill in mission, flagship project, outputs
- `RUNBOOK.md` — Fill in canonical paths, boot/closeout specifics
- `HANDOFF.md` — Initialize with "Department created, no work done yet"
- `STATE.json` — Leave as-is (initial state)

---

## Phase 5: Create Projects

For each project:

```bash
# Copy template
cp -r ORG/PROJECTS/example-project ORG/PROJECTS/<project-name>
```

### 5.1 Edit PIPELINE_STATE.json

This is the most important file. Replace all placeholders:

```json
{
  "project": "<project-name>",
  "config": {
    "notifyTopic": "<notification-channel-or-remove>",
    "roles": {
      "constitute": { "agentId": "<actual-agent-id>", "model": "<actual-model>" },
      "research":   { "agentId": "<actual-agent-id>", "model": "<actual-model>" },
      "specify":    { "agentId": "<actual-agent-id>", "model": "<actual-model>" },
      "plan":       { "agentId": "<actual-agent-id>", "model": "<actual-model>" },
      "implement":  { "agentId": "<actual-agent-id>", "model": "<actual-model>" },
      "test":       { "agentId": "<actual-agent-id>", "model": "<actual-model>" },
      "review":     { "agentId": "<actual-agent-id>", "model": "<actual-model>" },
      "gap_analysis": { "agentId": "<actual-agent-id>", "model": "<actual-model>" }
    },
    "escalation": {
      "chain": ["<cheap-model>", "<mid-model>", "<strong-model>"]
    },
    "peerConsult": {
      "consultModels": ["<model-a>", "<model-b>", "<model-c>"],
      "synthesizerModel": "<strongest-model>"
    }
  }
}
```

### 5.2 Edit STATUS.md
- Project name and description
- "Pipeline Run #1 pending"

### 5.3 Edit DECISIONS.md
- Add initial decision: "Adopted ORG Framework Pipeline for project management"

---

## Phase 6: Update Org-Level Files

### 6.1 TASKBOARD.md
Add the human's current priorities.

### 6.2 ASSET_REGISTRY.md
Register the Pipeline framework as an L1 asset:
```markdown
### Infrastructure
- L1: Pipeline Orchestrator Framework
  - Owner: <dept-name>
  - Where: `PROJECTS/pipeline-framework/`
```

---

## Phase 7: Clean Up Examples

```bash
rm -rf ORG/DEPARTMENTS/example-dept
rm -rf ORG/PROJECTS/example-project
```

---

## Phase 8: Create Orchestrator Cron Job

This is what drives the Pipeline automatically. Use the `cron` tool:

```json
{
  "name": "pipeline:<project-name>:orchestrator",
  "schedule": { "kind": "cron", "expr": "*/30 * * * *", "tz": "<human-timezone>" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "You are the Pipeline Orchestrator for project <project-name>.\n\nYour only job: read Pipeline state → check current phase → spawn sub-agents → verify artifacts → advance phases.\nYou do NOT do the actual work.\n\nRequired reads:\n- ORG/PROJECTS/<project-name>/PIPELINE_STATE.json\n- ORG/TASKBOARD.md\n- ORG/DEPARTMENTS/<dept>/HANDOFF.md\n- ORG/ASSET_REGISTRY.md\n\nPhase prompt templates: ORG/PROJECTS/pipeline-framework/templates/PHASE_PROMPTS/\nOrchestrator logic: ORG/PROJECTS/pipeline-framework/templates/ORCHESTRATOR_PROMPT.template.md\n\nFollow the orchestrator template exactly. Use sessions_spawn with explicit agentId and model from PIPELINE_STATE.json config.roles. Never omit the model parameter.\n\nAfter work: update PIPELINE_STATE.json + append PIPELINE_LOG.jsonl + update dept HANDOFF.md.",
    "model": "<cheap-model>",
    "timeoutSeconds": 1200
  },
  "delivery": {
    "mode": "announce",
    "channel": "<your-channel>",
    "to": "<notification-target>"
  }
}
```

🧑 Confirm with your human before creating the cron job.

---

## Phase 9: Wire Boot Sequence into Agent Config

Add to your workspace's `AGENTS.md` (or equivalent system prompt file):

```markdown
## ORG Boot Sequence (mandatory, every session)
Before doing any work, read in order:
1. `ORG/TASKBOARD.md` — current priorities
2. `ORG/DEPARTMENTS/<your-dept>/HANDOFF.md` — your team's state
3. `ORG/ASSET_REGISTRY.md` — reusable assets (reuse first, build second)

## ORG Closeout (mandatory, after work)
1. Update your dept `HANDOFF.md` (Done / Next / Blockers + links)
2. Update `ORG/PROJECTS/<project>/STATUS.md` if you advanced the project
3. All conclusions must be persisted to files — never just say "done" in chat
```

---

## Phase 10: Verify & Report

Run these checks:

- [ ] `ORG/` directory exists with all core files
- [ ] At least one department with filled CHARTER + HANDOFF
- [ ] At least one project with configured PIPELINE_STATE.json
- [ ] No `<placeholder>` strings remaining in any JSON/MD file
- [ ] Orchestrator cron job created and enabled
- [ ] Boot sequence added to AGENTS.md or system prompt

```bash
# Quick check for leftover placeholders
grep -rn "<your-\|<actual-\|<project-\|<dept-\|<cheap-\|<mid-\|<strong-\|<model-\|<notification-\|<human-\|<owner>" ORG/
```

Report to your human:
1. What departments and projects you created
2. The cron job config (name, schedule, model)
3. Any decisions you need from them
4. Suggested next step: "The Pipeline is ready. The orchestrator will trigger in ≤30 minutes and start Phase 0 (Constitute) for your project."

---

## Cleanup

```bash
rm -rf /tmp/org-framework
```

---

## Reference: File Purposes

| File | Who Writes | Who Reads | Purpose |
|------|-----------|-----------|---------|
| TASKBOARD.md | Human / main agent | All agents | Current priorities |
| HANDOFF.md | Each dept's agents | Next session / other depts | State continuity |
| PIPELINE_STATE.json | Orchestrator only | Orchestrator + sub-agents | Phase state machine |
| PIPELINE_LOG.jsonl | Orchestrator only | Anyone (audit) | Append-only history |
| pipeline/*.md | Sub-agents (one per phase) | Next phase's sub-agent | Phase artifacts |
| ASSET_REGISTRY.md | Asset owners | All agents | Reuse discovery |
| MEMORY_POLICY.md | Org admin | All agents | Rules for what goes where |
