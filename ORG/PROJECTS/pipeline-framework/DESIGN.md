# Pipeline Orchestrator 框架设计文档

> Author: Architect | Status: APPROVED

---

## 0. 背景与动机

### 现状问题
1. **扁平 cron 驱动**：每个 cron job = 一个 agent + 一次 isolated session，把调研/设计/编码/测试/review 全塞进一个 prompt，质量不可控。
2. **无阶段门控**：没有"上一步没做好就不能进下一步"的机制，agent 自行决定做多少。
3. **无多角色协作**：Chat bots cannot trigger each other (bot-to-bot messages blocked), so multi-role collaboration cannot happen at the messaging layer。
4. **上下文污染**：单次 session 内对话越长，上下文越脏，后期输出质量下降。

### 设计目标
- 引入阶段状态机（Pipeline），每个阶段有明确的准入条件、产出物、退出条件
- 每个阶段在干净的 isolated session 中执行，通过文件（而非对话）传递上下文
- 多角色协作发生在 OpenClaw 内部（sessions_spawn），messaging platforms serve only as result announcement panels
- 完全兼容现有 ORG 章程（Boot/Closeout/落盘/变更控制）
- 可复用：任何项目都能套用同一套 Pipeline 框架

---

## 1. 核心概念

### 1.1 Pipeline = 阶段状态机

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Phase 0  │───▶│ Phase 1  │───▶│ Phase 2  │───▶│ Phase 3  │───▶│ Phase 4  │───▶│ Phase 5  │───▶│ Phase 6  │───▶│ Phase 7  │
│Constitute│    │ Research │    │ Specify  │    │Plan+Tasks│    │Implement │    │  Test    │    │ Review   │    │Gap Analys│
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
      │                                                               │                              │                │
      │                                                               │         ┌───────────┐        │                │
      │                                                               │◀────────│ 回退重做   │◀───────│                │
      │                                                               │         └───────────┘        │                │
      ▼                                                                                              ▼                ▼
  CONSTITUTION.md                                                                              REVIEW_REPORT.md  GAP_ANALYSIS.md
                                                                                               → pass: 进入 Phase 7
                                                                                               → fail: 回退到指定阶段
```

### 1.2 Orchestrator = 调度 Agent

Orchestrator 不是 OpenClaw 内置工具，而是一个 agent 角色，由 cron 定时触发。它的职责：

1. 读 `PIPELINE_STATE.json` → 判断当前阶段
2. 检查当前阶段的准入条件是否满足
3. 用 `sessions_spawn` 启动对应角色的子 agent 执行当前阶段
4. 子 agent 完成后，检查产出物是否满足退出条件
5. 满足 → 推进到下一阶段；不满足 → 标记阻塞，等下次触发重试
6. 将阶段转换事件写入 `PIPELINE_LOG.jsonl`
7. broadcast progress summary to notification channel

### 1.3 角色分工

| 阶段 | 推荐角色 (agentId) | 推荐模型 | 职责 |
|------|-------------------|---------|------|
| Phase 0: Constitute | <your-architect-agent> | opus | 定义项目原则、约束、技术边界 |
| Phase 1: Research | <your-researcher-agent> | gpro | 调研现有方案、论文、开源实现/或者是进一步调研/背景分析 |
| Phase 2: Specify | <your-designer-agent> | opus | 需求规格、接口定义、验收标准 |
| Phase 3: Plan+Tasks | <your-architect-agent> | opus | 实现计划、任务分解、测试方案设计 |
| Phase 4: Implement | <your-coder-agent> | codex/glm/sonnet | 逐任务编码（每个任务一个干净 session） |
| Phase 5: Test | <your-coder-agent> | codex | 执行测试、生成测试报告 |
| Phase 6: Review | <your-reviewer-agent> | opus | 质量审查、目标达成分析、通过/回退决策 |
| Phase 7: Gap Analysis | <your-researcher-agent> | gpro | 差距分析、跨轮次追踪、下轮改进建议 |

> 注：角色和模型可按项目需求调整，以上为默认推荐。

---

## 2. 文件结构

### 2.1 项目级 Pipeline 目录

每个使用 Pipeline 的项目，在 `ORG/PROJECTS/<project>/` 下新增：

```
ORG/PROJECTS/<project>/
├── STATUS.md              # 已有 — 人类可读的项目状态
├── DECISIONS.md           # 已有 — 关键决策记录
├── RUNBOOK.md             # 已有 — 运行手册
├── PIPELINE_STATE.json    # 新增 — 阶段状态机（机器读写）
├── PIPELINE_LOG.jsonl     # 新增 — 全历史阶段转换日志（append-only，跨轮次）
├── pipeline/              # 新增 — 当前轮次产出物（固定路径，Orchestrator 直接读写）
│   ├── CONSTITUTION.md    # Phase 0 产出
│   ├── RESEARCH.md        # Phase 1 产出
│   ├── SPECIFICATION.md   # Phase 2 产出
│   ├── PLAN.md            # Phase 3 产出
│   ├── TASKS.md           # Phase 3 产出
│   ├── IMPL_STATUS.md     # Phase 4 进度追踪
│   ├── TEST_REPORT.md     # Phase 5 产出
│   └── REVIEW_REPORT.md   # Phase 6 产出
└── pipeline_archive/      # 新增 — 历史轮次归档（每轮 Review PASS 后自动归档）
    ├── run-001/           # 第 1 轮完整产出快照
    │   ├── CONSTITUTION.md
    │   ├── RESEARCH.md
    │   ├── SPECIFICATION.md
    │   ├── PLAN.md
    │   ├── TASKS.md
    │   ├── IMPL_STATUS.md
    │   ├── TEST_REPORT.md
    │   └── REVIEW_REPORT.md
    ├── run-002/
    └── ...
```

**归档机制**：
- `pipeline/` 始终代表"当前正在进行的这一轮"，路径固定，Orchestrator 和子 agent 无需关心轮次编号
- 每轮 Review PASS 后，Orchestrator 自动执行归档：
  1. 复制 `pipeline/*` → `pipeline_archive/run-{N}/`
  2. 在 PIPELINE_STATE.json 中 `runNumber++`，所有阶段重置为 pending
  3. 在 PIPELINE_LOG.jsonl 追加 `{"event": "run_archived", "run": N}`
- PIPELINE_LOG.jsonl 保持全局 append-only 不按轮次拆分，便于跨轮次趋势分析

### 2.2 PIPELINE_STATE.json Schema

```json
{
  "project": "example-project",
  "version": 1,
  "runNumber": 1,
  "currentPhase": "research",
  "phases": {
    "constitute": {
      "status": "done",
      "artifact": "pipeline/CONSTITUTION.md",
      "completedAt": "2026-02-13T10:00:00+08:00",
      "completedBy": "<your-architect-agent>"
    },
    "research": {
      "status": "in_progress",
      "artifact": "pipeline/RESEARCH.md",
      "startedAt": "2026-02-13T11:00:00+08:00",
      "assignedTo": "<your-researcher-agent>",
      "retryCount": 0
    },
    "specify": {
      "status": "pending",
      "artifact": "pipeline/SPECIFICATION.md"
    },
    "plan": {
      "status": "pending",
      "artifact": "pipeline/PLAN.md"
    },
    "implement": {
      "status": "pending",
      "artifact": "pipeline/IMPL_STATUS.md",
      "subtasks": []
    },
    "test": {
      "status": "pending",
      "artifact": "pipeline/TEST_REPORT.md"
    },
    "review": {
      "status": "pending",
      "artifact": "pipeline/REVIEW_REPORT.md"
    },
    "gap_analysis": {
      "status": "pending",
      "artifact": "pipeline/GAP_ANALYSIS.md"
    }
  },
  "blockers": [],
  "lastOrchestratorRun": "2026-02-13T11:00:00+08:00",
  "config": {
    "maxRetries": 3,
    "autoAdvance": true,
    "notifyTopic": "<your-notification-channel>",
    "roles": {
      "constitute": { "agentId": "<your-architect-agent>", "model": "opus" },
      "research":   { "agentId": "<your-researcher-agent>", "model": "gpro" },
      "specify":    { "agentId": "<your-designer-agent>",  "model": "opus" },
      "plan":       { "agentId": "<your-architect-agent>", "model": "opus" },
      "implement":  { "agentId": "<your-coder-agent>",     "model": "sonnet/codex/glm" },
      "test":       { "agentId": "<your-coder-agent>",     "model": "codex" },
      "review":     { "agentId": "<your-reviewer-agent>",  "model": "opus" },
      "gap_analysis":{ "agentId": "<your-researcher-agent>", "model": "gpro" }
    }
  }
}
```

### 2.3 PIPELINE_LOG.jsonl 格式

每行一条事件，append-only：

```jsonl
{"ts":"2026-02-13T10:00:00+08:00","event":"phase_complete","phase":"constitute","agent":"<your-architect-agent>","duration_s":120,"artifact":"pipeline/CONSTITUTION.md"}
{"ts":"2026-02-13T11:00:00+08:00","event":"phase_start","phase":"research","agent":"<your-researcher-agent>"}
{"ts":"2026-02-13T11:15:00+08:00","event":"phase_complete","phase":"research","agent":"<your-researcher-agent>","duration_s":900,"artifact":"pipeline/RESEARCH.md"}
{"ts":"2026-02-13T12:00:00+08:00","event":"review_reject","phase":"review","agent":"<your-reviewer-agent>","reason":"测试覆盖率不足","rollbackTo":"implement"}
```

---

## 3. 阶段详细定义

### Phase 0: Constitute（立宪）

**目的**：定义项目的基本原则、技术约束、质量标准。类似 SpecKit 的 CONSTITUTION。

**准入条件**：项目已在 `ORG/PROJECTS/` 下建立目录

**产出物**：`pipeline/CONSTITUTION.md`，包含：
- 项目目标（1-3 句话）
- 技术栈约束（语言、框架、依赖限制）
- 质量标准（测试覆盖率要求、性能指标、文档要求）
- 边界约束（不做什么、安全红线）
- 与 ORG 章程的对齐声明

**退出条件**：CONSTITUTION.md 存在且非空，包含以上所有章节

---

### Phase 1: Research（调研）

**目的**：调研现有方案、论文、开源实现，建立知识基础。

**准入条件**：Phase 0 完成

**产出物**：`pipeline/RESEARCH.md`，包含：
- 调研范围与方法
- 关键发现（至少 5 条，每条附来源链接）
- 现有方案对比表
- 技术风险识别
- 推荐方向（附理由）

**退出条件**：RESEARCH.md 存在，包含至少 5 条有来源的发现

---

### Phase 2: Specify（需求规格）

**目的**：基于调研结果，定义精确的需求规格和验收标准。借鉴 SpecKit 的 SPECIFICATION。

**准入条件**：Phase 1 完成

**输入**：CONSTITUTION.md + RESEARCH.md

**产出物**：`pipeline/SPECIFICATION.md`，包含：
- 功能需求列表（每条可测试）
- 非功能需求（性能、可靠性、可维护性）
- 接口定义（输入/输出格式）
- 验收标准（每个需求对应的验收条件）
- 排除项（明确不做什么）

**退出条件**：SPECIFICATION.md 存在，每个功能需求都有对应的验收标准

---

### Phase 3: Plan + Tasks（计划与任务分解）

**目的**：制定实现计划，分解为可执行的原子任务。借鉴 SpecKit 的 PLAN + TASKS。

**准入条件**：Phase 2 完成

**输入**：CONSTITUTION.md + RESEARCH.md + SPECIFICATION.md

**产出物**：
- `pipeline/PLAN.md`：实现路线图（分阶段、有依赖关系）
- `pipeline/TASKS.md`：原子任务列表，每个任务包含：
  - 任务 ID（T-001, T-002...）
  - 描述
  - 依赖（哪些任务必须先完成）
  - 预期产出文件
  - 测试方案（如何验证这个任务完成了）
  - 预估复杂度（S/M/L）

**退出条件**：PLAN.md + TASKS.md 存在，每个任务都有测试方案

---

### Phase 4: Implement（实现）

**目的**：逐任务编码。每个任务在独立的干净 session 中执行。

**准入条件**：Phase 3 完成

**输入**：每个子任务只注入 CONSTITUTION.md + SPECIFICATION.md + 该任务在 TASKS.md 中的描述 + 相关代码文件

**执行方式**：
- Orchestrator 从 TASKS.md 中按依赖顺序取出下一个未完成任务
- `sessions_spawn` 一个子 agent，只给它该任务需要的最小上下文
- 子 agent 完成后更新 `pipeline/IMPL_STATUS.md`
- 每次 Orchestrator 触发处理 1-3 个任务（避免超时）

**产出物**：
- 代码文件（在项目 repo 中）
- `pipeline/IMPL_STATUS.md`：任务完成状态追踪

**退出条件**：TASKS.md 中所有任务标记为 done

---

### Phase 5: Test（测试）

**目的**：执行分级测试，生成测试报告。

**准入条件**：Phase 4 完成（所有任务 done）

**测试分级**：
1. **单元测试**：每个任务的独立测试
2. **集成测试**：跨任务的接口测试
3. **验收测试**：对照 SPECIFICATION.md 的验收标准逐条验证

**产出物**：`pipeline/TEST_REPORT.md`，包含：
- 测试执行摘要（通过/失败/跳过数量）
- 每条验收标准的通过状态
- 失败用例详情
- 测试覆盖率（如适用）

**退出条件**：TEST_REPORT.md 存在，验收测试通过率 >= 阈值（默认 80%，可在 config 中调整）

---

### Phase 6: Review（审查）

**目的**：质量审查 + 目标达成分析。这是最关键的门控。

**准入条件**：Phase 5 完成

**输入**：全部 pipeline 产出物

**审查维度**：
1. **规格符合度**：代码是否满足 SPECIFICATION.md 的所有需求？
2. **质量标准**：是否满足 CONSTITUTION.md 定义的质量标准？
3. **测试充分性**：TEST_REPORT.md 是否覆盖了所有关键路径？
4. **可维护性**：代码结构、文档、注释是否足够？
5. **目标达成**：整体是否达到了项目目标？

**产出物**：`pipeline/REVIEW_REPORT.md`，包含：
- 各维度评分（1-5）
- 总体判定：`PASS` / `FAIL`
- 如果 FAIL：指出具体问题 + 建议回退到哪个阶段
- 如果 PASS：项目完成确认

**退出条件**：
- PASS → Pipeline 完成，更新 `ORG/PROJECTS/<project>/STATUS.md`
- FAIL → Orchestrator 将 currentPhase 回退到指定阶段，附上 review 意见作为该阶段的额外输入

---

### Phase 7: Gap Analysis（差距分析）

**目的**：深度分析当前轮次成果与项目最终目标之间的差距，为下一轮 Pipeline 提供结构化的改进方向。Review 回答"这轮过不过"，Gap Analysis 回答"下轮怎么更好"。

**准入条件**：Phase 6 Review 判定为 PASS

**输入**：全部 pipeline 产出物 + CONSTITUTION.md（最终目标基准）+ 历史轮次归档（如有）

**产出物**：`pipeline/GAP_ANALYSIS.md`，包含：
- **量化完成度**：逐模块评估当前成果与最终目标的差距（百分比或评分）
- **工况/场景覆盖分析**：已覆盖哪些场景、缺失哪些边界/极端场景
- **图表充分性评估**：现有图表是否足以支撑结论，建议新增哪些
- **跨轮次进步追踪**：与上一轮（如有）的关键指标对比，量化改进幅度
- **下轮改进建议**：按优先级（高/中/低）列出具体可执行的改进项，每项附理由和预期收益
- **质量标准更新建议**：是否需要调整验收阈值或新增质量维度

**退出条件**：GAP_ANALYSIS.md 存在且非空，包含量化完成度和至少 3 条分优先级的改进建议

**角色配置**：
- 推荐 agentId：`<your-researcher-agent>`
- 推荐模型：`gpro`（擅长长文深度分析和结构化输出）

**与 Review 的区别**：
- Review（Phase 6）是门控——决定 PASS/FAIL，关注"这轮做得够不够好"
- Gap Analysis（Phase 7）是前瞻——假设已 PASS，关注"下轮怎么做得更好"
- Review 由 reviewer 角色执行（严格把关），Gap Analysis 由 professor 角色执行（深度分析）

---

## 4. Orchestrator 运行逻辑（伪代码）

```
每次 cron 触发：

1. 读 PIPELINE_STATE.json
2. current = state.currentPhase

3. if current.status == "pending":
     # 启动该阶段
     检查准入条件（前置阶段的 artifact 是否存在且有效）
     if 准入条件不满足:
       记录 blocker，播报，退出
     role = state.config.roles[current]
     sessions_spawn(agentId=role.agentId, model=role.model, task=构建阶段 prompt)
     更新 status → "in_progress"

4. if current.status == "in_progress":
     # 检查产出物
     if artifact 存在且满足退出条件:
       更新 status → "done"
       推进 currentPhase → 下一阶段
       写 PIPELINE_LOG.jsonl
       broadcast progress to notification channel
     elif retryCount >= maxRetries:
       标记 blocker，通知人工介入
     else:
       retryCount++
       重新 spawn 该阶段

5. if current == "review" && status == "done":
     读 REVIEW_REPORT.md
     if 判定 == PASS:
       推进 currentPhase → gap_analysis（pending）
     if 判定 == FAIL:
       回退 currentPhase 到指定阶段
       将 review 意见注入该阶段的额外输入

5b. if current == "gap_analysis" && status == "done":
     Pipeline 本轮完成 🎉
     归档：复制 pipeline/* → pipeline_archive/run-{runNumber}/
     更新 ORG/PROJECTS/<project>/STATUS.md
     PIPELINE_LOG.jsonl 追加 {"event": "run_archived", "run": runNumber}
     runNumber++，所有阶段重置为 pending（准备下一轮）
     下一轮 Constitution 阶段自动获得 GAP_ANALYSIS.md 作为输入

6. 保存 PIPELINE_STATE.json
```

---

## 5. Orchestrator Cron Job 模板

```json
{
  "name": "pipeline:<project>:orchestrator",
  "schedule": { "kind": "cron", "expr": "*/30 * * * *", "tz": "Asia/Shanghai" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "你是 Pipeline Orchestrator。\n\n你的唯一职责是推进项目 pipeline，不要自己做具体工作。\n\n必读文件：\n- Pipeline 状态：ORG/PROJECTS/<project>/PIPELINE_STATE.json\n- ORG Boot Sequence：先读 TASKBOARD.md → 部门 HANDOFF.md → ASSET_REGISTRY.md\n\n执行逻辑：\n1. 读 PIPELINE_STATE.json，确定 currentPhase 和 runNumber\n2. 检查当前阶段的准入条件（前置 artifact 是否存在）\n3. 如果阶段 pending → sessions_spawn 对应角色执行该阶段\n4. 如果阶段 in_progress → 检查产出物是否满足退出条件\n5. 满足 → 推进到下一阶段；不满足 → 重试或标记 blocker\n6. 如果 Review PASS → 归档当前轮次（复制 pipeline/* → pipeline_archive/run-{N}/），runNumber++，所有阶段重置为 pending\n7. 如果 Review FAIL → 回退到指定阶段，注入 review 意见\n8. 更新 PIPELINE_STATE.json + 追加 PIPELINE_LOG.jsonl\n9. broadcast progress summary to notification channel\n\n硬约束：\n- 不要自己写代码/调研/测试，全部通过 sessions_spawn 委派\n- 每次触发最多推进 1 个阶段\n- 不修改系统配置/网关\n- 遵守 ORG Closeout：更新部门 HANDOFF.md",
    "model": "gflash2",
    "timeoutSeconds": 600
  },
  "delivery": {
    "mode": "announce",
    "channel": "<your-channel>",
    "to": "<your-notification-channel>"
  }
}
```

> Orchestrator 本身用轻量模型（gflash2/mini），因为它只做判断和调度，不做重活。

---

## 6. 多角色协作机制

### 6.1 内部协作（sessions_spawn）

```
Orchestrator
  ├── spawn(<your-researcher-agent>, "调研 XX 领域...")  → RESEARCH.md
  ├── spawn(<your-architect-agent>, "设计 XX 架构...")   → PLAN.md
  ├── spawn(<your-coder-agent>, "实现任务 T-003...")     → code
  └── spawn(<your-reviewer-agent>, "审查 pipeline...")   → REVIEW_REPORT.md
```

每个 spawn 的子 agent：
- 在独立 isolated session 中运行
- 只接收该阶段需要的文件路径作为输入
- 产出物写入 `pipeline/` 目录
- 完成后 session 自动结束

### 6.2 外部播报（Notification Channel）

Orchestrator broadcasts a summary to the project notification channel on each phase transition:

```
📋 Pipeline [example-project] 进度更新
━━━━━━━━━━━━━━━━━━━━━━
✅ Phase 0: Constitute — 完成
✅ Phase 1: Research — 完成
🔄 Phase 2: Specify — 进行中 (by @openclaw_designer_bot)
⬜ Phase 3: Plan+Tasks
⬜ Phase 4: Implement
⬜ Phase 5: Test
⬜ Phase 6: Review
━━━━━━━━━━━━━━━━━━━━━━
下次检查：30 分钟后
```

### 6.3 人工介入点

以下情况 Orchestrator 会暂停并通知 CEO：
- 某阶段重试 3 次仍未通过
- Review 阶段判定 FAIL 且需要回退超过 2 个阶段
- 遇到需要系统级变更的 blocker
- 项目完成（PASS）

---

## 7. 与现有 ORG 章程的兼容性对照

| ORG 规则 | Pipeline 如何遵守 |
|---------|-----------------|
| Boot Sequence（读 TASKBOARD → HANDOFF → ASSET_REGISTRY） | Orchestrator prompt 中强制要求；每个子 agent 的 prompt 中也注入 |
| Closeout（更新 HANDOFF/STATUS） | Orchestrator 每次阶段转换后自动更新 |
| 落盘规则（不允许只在聊天里说做完了） | 所有阶段产出物都是文件，PIPELINE_LOG.jsonl 记录全过程 |
| 变更控制（系统配置只有保障部能改） | Orchestrator 和子 agent 都不碰系统配置 |
| 记忆分层（公司/部门/项目/agent） | Pipeline 产出物在项目级，状态在项目级，不污染其他层 |
| 复用制度（L0/L1/L2） | Pipeline 框架本身作为 L1 资产登记 |

---

## 8. 可观测性（Observability）

借鉴 Effective Harnesses 文章的建议：

### 8.1 PIPELINE_LOG.jsonl
- 每次阶段转换记录一条（时间、事件类型、agent、耗时、产出物路径）
- Append-only，不可修改，用于事后审计

### 8.2 阶段 Checkpoint
- 每个阶段完成时，在 PIPELINE_STATE.json 中记录 completedAt、completedBy、duration
- 如果阶段内部有多步（如 implement 的多个子任务），在 IMPL_STATUS.md 中逐任务追踪

### 8.3 Guardrails（护栏）
- 准入条件检查：前置 artifact 必须存在且非空
- 退出条件检查：产出物必须满足最低质量标准
- 重试上限：默认 3 次，超过则人工介入
- 超时保护：每个子 agent 有 timeoutSeconds 限制

---

## 9. 灵活性设计

### 9.1 阶段可裁剪
不是所有项目都需要 7 个阶段。小项目可以跳过：
- 跳过 Phase 0（用部门 CHARTER 代替）
- 合并 Phase 1+2（调研和规格一起做）
- 跳过 Phase 6（小改动不需要正式 review）

在 PIPELINE_STATE.json 的 phases 中，将不需要的阶段设为 `"status": "skipped"` 即可。

### 9.2 角色可替换
`config.roles` 中的 agentId 和 model 可按项目调整。比如纯研究项目可以让 professor 做更多阶段。

### 9.3 触发频率可调
Orchestrator 的 cron 频率可以从 5 分钟到 24 小时不等，取决于项目紧迫程度。

---

## 10. 与现有 Cron Job 的迁移关系

现有的单体 cron job（如 example:iteration-loop）不需要立即废弃。迁移策略：

1. **新项目**：直接使用 Pipeline 框架
2. **现有项目**：在当前迭代周期结束后，创建 Pipeline 目录，将已有产出物映射到对应阶段，然后切换到 Pipeline 模式
3. **维护类 cron**（如 example-maintenance）：不需要 Pipeline，保持现状。Pipeline 适用于有明确目标和交付物的项目

---

## 附录 A：术语表

| 术语 | 含义 |
|------|------|
| Pipeline | 项目的阶段状态机，定义了从立项到交付的完整流程 |
| Orchestrator | 调度 agent，负责读取 Pipeline 状态并推进阶段 |
| Phase | Pipeline 中的一个阶段 |
| Artifact | 阶段的产出物（文件） |
| Gate | 阶段之间的门控条件（准入/退出） |
| Spawn | 通过 sessions_spawn 启动一个子 agent session |
| Rollback | Review 不通过时，将 Pipeline 回退到指定阶段 |

---

## 11. 双层求助机制（Assistance Protocol）

> 新增于 2026-02-14 | 解决子 agent 卡住时无横向求助通道的问题

### 11.1 问题场景

子 agent 在执行阶段任务时可能因以下原因卡住：
- 模型能力不足（简单模型解不了复杂问题）
- 需要不同视角（同一个模型反复尝试同一思路）
- 参数调优类任务需要多方案对比

现有机制只有 `maxRetries` 重试同一模型，没有升级或求助通道。

### 11.2 第一层：Model Escalation（模型升级链）

当子 agent 执行失败时，Orchestrator 自动沿预定义的升级链换更强模型重试：

```
mini → glm → codex → sonnet → ⛔ 人工介入
```

配置（在 PIPELINE_STATE.json 的 config 中）：

```json
"escalation": {
  "enabled": true,
  "chain": ["mini", "glm", "codex", "sonnet"],
  "escalateAfterFails": 1,
  "humanThreshold": "sonnet"
}
```

- `chain`：模型升级顺序，从便宜到贵
- `escalateAfterFails`：每个模型失败几次后升级（默认 1）
- `humanThreshold`：到达此模型仍失败则触发第二层或人工介入

逻辑：
1. 初始模型（由 config.roles 指定）执行失败
2. Orchestrator 取 chain 中下一个模型，重新 spawn
3. 每次升级记录到 PIPELINE_LOG.jsonl：`{"event":"model_escalated","fromModel":"...","toModel":"..."}`
4. 到达 humanThreshold 仍失败 → 进入第二层

### 11.3 第二层：Peer Consult（并行多模型求助）

同时向多个不同模型发起咨询，收集多角度方案，综合后重试。

配置：

```json
"peerConsult": {
  "enabled": true,
  "triggerAfterEscalationFails": 2,
  "consultModels": ["gpro", "glm", "sonnet"],
  "consultTimeout": 300,
  "synthesizerModel": "opus",
  "maxConsultRounds": 1
}
```

流程：

```
子 agent 卡住（escalation 链耗尽）
    │
    ▼
Orchestrator 收集错误上下文
    │
    ├── spawn(consultant, model=gpro,   task=consult_request prompt)
    ├── spawn(consultant, model=glm,    task=consult_request prompt)
    └── spawn(consultant, model=sonnet, task=consult_request prompt)
    │
    ▼ （三个顾问并行返回）
    │
spawn(synthesizer, model=opus, task=consult_synthesize prompt)
    │
    ▼
将综合方案注入原阶段 prompt
    │
    ▼
用 escalation chain 最强模型重新 spawn 原任务
    │
    ├─ 成功 → 正常推进 ✅
    └─ 失败 → 标记 blocker，通知人工 🚨
```

### 11.4 子 Agent 卡住报告协议

子 agent 在 Phase prompt 中被告知：连续尝试 2 次失败后，不要继续死磕，在产出文件中报告 stuck 状态：

Implement 阶段（IMPL_STATUS.md）：
```
- T-xxx: stuck | 错误摘要: <一句话> | 已尝试: <方案列表> | 相关文件: <路径>
```

Test 阶段（TEST_REPORT.md）：
```
### Stuck Items
- 测试项: <FR-xxx> | 错误摘要: <一句话> | 已尝试: <方案列表> | 相关文件: <路径>
```

Orchestrator 检测到 stuck 标记后自动进入求助流程。

### 11.5 PIPELINE_STATE.json 扩展字段

阶段对象新增 `stuckInfo`：

```json
{
  "implement": {
    "status": "stuck",
    "stuckInfo": {
      "taskId": "T-003",
      "failCount": 4,
      "errorSummary": "PyBaMM Chen2020 模型参数不收敛",
      "escalationLevel": 3,
      "consultRequested": true,
      "consultResults": [
        {"model": "gpro", "sessionKey": "...", "status": "done", "suggestion": "..."},
        {"model": "glm",  "sessionKey": "...", "status": "done", "suggestion": "..."},
        {"model": "sonnet","sessionKey": "...", "status": "done", "suggestion": "..."}
      ],
      "synthesizedSolution": "综合方案：...",
      "retryWithSolution": false
    }
  }
}
```

### 11.6 新增日志事件类型

```jsonl
{"ts":"...","event":"model_escalated","phase":"implement","task":"T-003","fromModel":"codex","toModel":"sonnet"}
{"ts":"...","event":"consult_requested","phase":"implement","task":"T-003","models":["gpro","glm","sonnet"]}
{"ts":"...","event":"consult_complete","phase":"implement","task":"T-003","model":"gpro"}
{"ts":"...","event":"solution_synthesized","phase":"implement","task":"T-003","synthesizer":"opus"}
{"ts":"...","event":"retry_with_solution","phase":"implement","task":"T-003","model":"sonnet"}
{"ts":"...","event":"human_escalation","phase":"implement","task":"T-003","reason":"所有自动求助均失败"}
```

### 11.7 成本控制

- 第一层升级链从最便宜的模型开始，逐步升级，避免一上来就用贵模型
- 第二层 Peer Consult 的顾问用中等模型（gpro/glm/sonnet），只有综合环节用 opus
- `maxConsultRounds` 限制求助轮数（默认 1 轮），防止无限循环
- 到达 humanThreshold 后不再自动升级到 opus 执行，而是等人类决定

### 11.8 Prompt 模板

新增两个模板文件：
- `templates/PHASE_PROMPTS/consult_request.md` — 顾问 agent 的 prompt
- `templates/PHASE_PROMPTS/consult_synthesize.md` — 方案综合 agent 的 prompt

详见模板文件内容。
