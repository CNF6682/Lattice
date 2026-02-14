# 🚀 Quick Deploy Guide — For Humans

> You have an OpenClaw instance running. You want your agent to set up the ORG Framework automatically.
> Just copy the prompt below and send it to your agent.

---

## Prerequisites

- OpenClaw running with at least one agent
- Agent has file read/write access to the workspace
- (Optional) Multiple agents configured if you want multi-role Pipeline

---

## Step 1: Send This Prompt to Your Agent

Copy everything inside the code block below and paste it as a message to your main agent:

````
I want you to deploy the ORG Framework to our workspace. This is a file-based multi-agent organization system with an 8-phase Pipeline for project management.

Here's what to do:

1. Clone the repo:
   git clone https://github.com/<owner>/org-framework.git /tmp/org-framework

2. Copy the ORG directory into our workspace:
   cp -r /tmp/org-framework/ORG <your-workspace-path>/ORG

3. Read the following files to understand the framework:
   - ORG/ORG_README.md (entrypoint)
   - ORG/MEMORY_POLICY.md (memory layering)
   - ORG/PIPELINE_GUIDE.md (pipeline overview)
   - ORG/PROJECTS/pipeline-framework/DESIGN.md (full design doc)

4. Set up our organization:
   a. Create a department for our work:
      - Copy ORG/DEPARTMENTS/example-dept/ to ORG/DEPARTMENTS/<our-dept-name>/
      - Fill in CHARTER.md with our mission and scope
      - Initialize HANDOFF.md with current state
   b. Create our first project:
      - Copy ORG/PROJECTS/example-project/ to ORG/PROJECTS/<our-project-name>/
      - Edit PIPELINE_STATE.json:
        - Replace all <your-xxx-agent> with our actual agent IDs
        - Replace all <your-xxx-model> with our actual model names
        - Set notifyTopic to our notification channel (or remove if not needed)
      - Fill in STATUS.md with project description
   c. Update ORG/TASKBOARD.md with our current priorities
   d. Remove the example-dept and example-project directories

5. Set up the Pipeline Orchestrator cron job:
   - Use the cron template from ORG/PROJECTS/pipeline-framework/DESIGN.md §5
   - Adjust: project name, agent IDs, model, cron schedule, notification channel
   - Create the cron job

6. Add Boot/Closeout instructions to our AGENTS.md (or system prompt):
   "Every session, before doing work: read ORG/TASKBOARD.md → your dept HANDOFF.md → ASSET_REGISTRY.md.
    After work: update HANDOFF.md and project STATUS.md."

7. Clean up: rm -rf /tmp/org-framework

After setup, give me a summary of what you created and the cron job config.
````

---

## Step 2: Review What the Agent Created

After the agent finishes, check:

- [ ] `ORG/` directory exists in your workspace
- [ ] At least one department under `DEPARTMENTS/` with filled-in CHARTER
- [ ] At least one project under `PROJECTS/` with configured PIPELINE_STATE.json
- [ ] Orchestrator cron job is running (`openclaw cron list` or ask your agent)
- [ ] TASKBOARD.md has your current priorities

---

## Step 3: Customize (Optional)

Things you might want to tweak:

| Setting | Where | Default |
|---------|-------|---------|
| Pipeline cron frequency | Cron job schedule | Every 30 min |
| Phase models | PIPELINE_STATE.json → config.roles | Varies |
| Escalation chain | PIPELINE_STATE.json → config.escalation | 3-tier |
| Max retries per phase | PIPELINE_STATE.json → config.maxRetries | 3 |
| Peer consult models | PIPELINE_STATE.json → config.peerConsult | 3 models |
| Skip phases | PIPELINE_STATE.json → phases.X.status | Set to "skipped" |

---

## Multi-Agent Setup

If you have multiple agents (architect, researcher, coder, reviewer), the Pipeline works best with role separation:

| Phase | Recommended Role | Why |
|-------|-----------------|-----|
| 0 Constitute | Architect / strong model | Sets the foundation |
| 1 Research | Researcher / long-context model | Needs deep analysis |
| 2 Specify | Designer / strong model | Precision matters |
| 3 Plan | Architect / strong model | Structural thinking |
| 4 Implement | Coder / fast coding model | Volume work |
| 5 Test | Coder / fast coding model | Execution-heavy |
| 6 Review | Reviewer / strong model | Quality gate |
| 7 Gap Analysis | Researcher / long-context model | Deep analysis |

Single-agent setups work too — just use the same agent ID for all roles and vary the model.

---

## Troubleshooting

**Agent doesn't follow Boot Sequence?**
→ Add it to AGENTS.md or the agent's system prompt explicitly.

**Pipeline stuck on a phase?**
→ Check PIPELINE_STATE.json for `retryCount` and `blockers`. Increase `maxRetries` or manually set the phase to `pending` to retry.

**Orchestrator not triggering?**
→ Verify the cron job exists and is enabled: ask your agent to run `cron list`.

**Want to restart a Pipeline run?**
→ Set all phases to `"status": "pending"` in PIPELINE_STATE.json. The orchestrator will pick it up next cycle.
