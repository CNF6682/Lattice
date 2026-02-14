# Phase 7: Gap Analysis（差距分析）

你正在执行项目 `<project>` 的 Pipeline Phase 7: Gap Analysis。

## 目的
深度分析当前轮次成果与项目最终目标之间的差距，为下一轮 Pipeline 提供结构化的改进方向。
Review 回答"这轮过不过"，你回答"下轮怎么更好"。

## 你需要读的文件（全部）
- `ORG/PROJECTS/<project>/pipeline/CONSTITUTION.md`（项目原则与最终目标基准）
- `ORG/PROJECTS/<project>/pipeline/RESEARCH.md`（调研报告）
- `ORG/PROJECTS/<project>/pipeline/SPECIFICATION.md`（需求规格与验收标准）
- `ORG/PROJECTS/<project>/pipeline/PLAN.md`（实现计划）
- `ORG/PROJECTS/<project>/pipeline/TASKS.md`（任务列表）
- `ORG/PROJECTS/<project>/pipeline/IMPL_STATUS.md`（实现状态）
- `ORG/PROJECTS/<project>/pipeline/TEST_REPORT.md`（测试报告）
- `ORG/PROJECTS/<project>/pipeline/REVIEW_REPORT.md`（审查报告——本轮评分与亮点）
- 历史轮次归档 `ORG/PROJECTS/<project>/pipeline_archive/`（如有，用于跨轮次对比）
- 项目 repo 中的代码文件（抽查关键模块）

## 分析维度

### 1. 量化完成度
- 逐模块评估当前成果与 CONSTITUTION.md 最终目标的差距
- 用百分比或评分量化每个模块的完成程度
- 明确标注哪些目标已达成、哪些部分达成、哪些尚未开始

### 2. 工况/场景覆盖分析
- 已覆盖哪些场景和工况
- 缺失哪些边界场景、极端工况、异常路径
- 哪些场景的覆盖深度不足（只有 happy path，缺少 edge case）

### 3. 图表与可视化充分性
- 现有图表是否足以支撑结论
- 建议新增哪些图表（附具体描述：X 轴、Y 轴、数据来源）
- 现有图表的改进建议（如需要）

### 4. 跨轮次进步追踪
- 与上一轮（如有归档）的关键指标对比
- 量化改进幅度（测试通过率、覆盖率、评分变化等）
- 识别停滞或退步的领域

### 5. 下轮改进建议
按优先级（高/中/低）列出具体可执行的改进项，每项包含：
- 改进内容描述
- 理由（为什么重要）
- 预期收益（量化或定性）
- 建议在哪个阶段重点关注

### 6. 质量标准更新建议
- 是否需要调整验收阈值
- 是否需要新增质量维度
- CONSTITUTION.md 是否需要更新（如项目方向微调）

## 你需要产出的文件
写入路径：`ORG/PROJECTS/<project>/pipeline/GAP_ANALYSIS.md`

必须包含以下章节：
1. **分析摘要**（一段话总结本轮差距全貌）
2. **量化完成度表**
   ```
   | 模块/目标 | 完成度 | 说明 |
   |-----------|--------|------|
   | ...       | XX%    | ...  |
   ```
3. **场景覆盖矩阵**（已覆盖 ✅ / 部分覆盖 ⚠️ / 未覆盖 ❌）
4. **跨轮次对比**（如有历史数据）
5. **改进建议清单**（至少 3 条，按优先级排序）
   ```
   | 优先级 | 改进项 | 理由 | 预期收益 | 关注阶段 |
   |--------|--------|------|----------|----------|
   | 高     | ...    | ...  | ...      | ...      |
   ```
6. **质量标准更新建议**（如无则写"当前标准适用，无需调整"）

## 完成标准
- GAP_ANALYSIS.md 存在且非空
- 包含量化完成度评估
- 包含至少 3 条分优先级的改进建议
- 每条建议附理由和预期收益

## 约束
- 不要修改任何 pipeline 文件或代码文件（你是分析者，不是修改者）
- 不要修改系统配置/网关
- 假设 Review 已 PASS——你的任务不是重新审判，而是前瞻性分析
- 完成后输出简短摘要（3-5 行）
