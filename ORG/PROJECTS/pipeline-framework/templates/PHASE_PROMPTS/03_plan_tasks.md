# Phase 3: Plan + Tasks（计划与任务分解）

你正在执行项目 `<project>` 的 Pipeline Phase 3: Plan + Tasks。

## 目的
制定实现计划，将需求分解为可执行的原子任务。每个任务必须足够小，能在一个独立 session 中完成。

## 你需要读的文件
- `ORG/PROJECTS/<project>/pipeline/CONSTITUTION.md`（项目原则与约束）
- `ORG/PROJECTS/<project>/pipeline/RESEARCH.md`（调研报告）
- `ORG/PROJECTS/<project>/pipeline/SPECIFICATION.md`（需求规格）

## 你需要产出的文件

### 文件 1：`ORG/PROJECTS/<project>/pipeline/PLAN.md`
实现路线图，包含：
1. **实现阶段划分**（按功能模块或依赖顺序分阶段）
2. **阶段间依赖关系**（哪些可以并行，哪些必须串行）
3. **技术方案选型**（基于 RESEARCH.md 的推荐，做最终技术决策）
4. **风险缓解计划**（针对 RESEARCH.md 识别的风险）

### 文件 2：`ORG/PROJECTS/<project>/pipeline/TASKS.md`
原子任务列表，每个任务包含：
- **任务 ID**（T-001, T-002...）
- **描述**（一句话说清楚要做什么）
- **依赖**（哪些任务必须先完成，用任务 ID 引用）
- **对应需求**（引用 SPECIFICATION.md 中的 FR-xxx）
- **预期产出文件**（代码文件路径或文档路径）
- **测试方案**（如何验证这个任务完成了——命令、断言、或人工检查项）
- **预估复杂度**（S: <30min / M: 30-60min / L: 60-120min）

## 完成标准
- PLAN.md + TASKS.md 都存在且非空
- 每个 SPECIFICATION.md 中的功能需求至少被一个任务覆盖
- 每个任务都有测试方案
- 任务依赖关系无环
- 复杂度 L 的任务不超过总数的 30%（如果超过，需要进一步拆分）

## 约束
- 不要修改前置阶段的文件
- 不要修改系统配置/网关
- 产出物必须落盘到指定路径
- 完成后输出简短摘要（3-5 行）

## 如果有 Review 回退意见
<review_feedback>
无
</review_feedback>
