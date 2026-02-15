# LATTICE

<div align="center">
  <img src="logo.png" width="120" alt="Lattice Logo" />
  <h3>The Operating System for Agent Teams</h3>
  <p>
    File-based organization. 8-phase pipelines. Zero chat entropy.
  </p>
  <p>
    <a href="#-quick-start">Quick Start</a> •
    <a href="#-philosophy">Philosophy</a> •
    <a href="#-architecture">Architecture</a>
  </p>
</div>

---

## ⚡️ Quick Start

**Don't configure files manually.** You have an Agent for that.

Copy the block below, fill in the `[CONFIG]` section, and paste it to your Agent.

```markdown
# 🏛️ LATTICE: ORG BOOTSTRAP PROTOCOL

You are the Chief of Staff. Your task is to initialize the Lattice Organization Framework in this workspace.

[CONFIG]
ORG_NAME="My Studio"
DEPARTMENTS=["Engineering", "Design"]
PRIMARY_PROJECT="Project Alpha"
ORCHESTRATOR_MODEL="gemini-1.5-flash"  # Low-cost coordinator
WORKER_MODEL="claude-3-5-sonnet"       # High-iq worker
[/CONFIG]

[EXECUTION]
1. CLONE: `git clone https://github.com/openclaw/lattice.git /tmp/lattice`
2. INSTALL: `cp -r /tmp/lattice/ORG ./ORG`
3. CONFIGURE:
   - Create directories for each item in DEPARTMENTS under `ORG/DEPARTMENTS/`
   - Create `ORG/PROJECTS/$PRIMARY_PROJECT` from template
   - Update `ORG/ORG_README.md` with ORG_NAME
   - Generate a valid `PIPELINE_STATE.json` using the specified models
4. CLEANUP: `rm -rf /tmp/lattice`
5. REPORT: List the new structure and next steps for the human.
```

---

## 💎 Philosophy

Chat history is **volatile**. Files are **persistent**.

Lattice moves Agent cognition out of the chat window and into the filesystem. It turns "talking to a bot" into "running an organization."

| Feature | Without Lattice | With Lattice |
| :--- | :--- | :--- |
| **Memory** | Scrolls away | Layered in `ORG/MEMORY/` |
| **State** | Implicit | Explicit in `STATUS.md` |
| **Handoffs** | Lost in context | Defined in `HANDOFF.md` |
| **Quality** | "Looks good to me" | 8-Phase Pipeline Gates |

---

## 🏗️ Architecture

### 1. The Organization Layer
Your workspace becomes a structured headquarters.

```
ORG/
├── ORG_README.md       # The Constitution
├── TASKBOARD.md        # The Global Queue
├── DEPARTMENTS/        # Specialized Agent Teams
│   ├── Engineering/    # Code, Tech Debt
│   └── Research/       # Analysis, Strategy
└── PROJECTS/           # Active Initiatives
    └── Alpha/          # Project "Alpha"
```

### 2. The 8-Phase Pipeline
Projects don't just "happen." They flow through a strict state machine.

`Constitute` → `Research` → `Specify` → `Plan` → `Implement` → `Test` → `Review` → `Gap Analysis`

*   **Orchestrator Agent**: Wakes up every 30m, checks phase, assigns work.
*   **Worker Agent**: Spawns, reads phase artifacts, executes, updates state, dies.
*   **Artifacts**: Every phase produces a tangible file (e.g., `REQUIREMENTS.md`).

---

## 📂 Directory Reference

*   `ORG/ORG_README.md` - **Start Here.** The root of trust.
*   `ORG/TASKBOARD.md` - High-level priorities.
*   `ORG/MEMORY_POLICY.md` - Rules for where information lives.
*   `ORG/PIPELINE_GUIDE.md` - The manual for your Agents.
*   `ORG/ASSET_REGISTRY.md` - Reusable tools and code blocks.

---

<div align="center">
  <sub>Built for <b>OpenClaw</b>. Compatible with any file-accessing Agent.</sub>
</div>
