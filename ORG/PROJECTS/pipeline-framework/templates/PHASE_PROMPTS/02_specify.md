# Phase 2: Specify（需求规格）

你正在执行项目 `<project>` 的 Pipeline Phase 2: Specify。

## 目的
基于调研结果，定义精确的需求规格和验收标准。这是"做什么"的权威定义，后续实现和测试都以此为准。

## 你需要读的文件
- `ORG/PROJECTS/<project>/pipeline/CONSTITUTION.md`（项目原则与约束）
- `ORG/PROJECTS/<project>/pipeline/RESEARCH.md`（调研报告）

## 你需要产出的文件
写入路径：`ORG/PROJECTS/<project>/pipeline/SPECIFICATION.md`

必须包含以下章节：
1. **功能需求列表**（每条需求有唯一 ID，如 FR-001，且可测试）
2. **非功能需求**（性能、可靠性、可维护性、兼容性）
3. **接口定义**（输入/输出格式、API 契约、数据结构）
4. **验收标准**（每个功能需求对应至少一条验收条件，格式：Given/When/Then 或等效描述）
5. **排除项**（明确不做什么，与 CONSTITUTION.md 的边界约束对齐）

## 完成标准
- SPECIFICATION.md 存在且非空
- 每个功能需求都有对应的验收标准
- 需求与 CONSTITUTION.md 的约束不矛盾
- 需求与 RESEARCH.md 的推荐方向一致

## 约束
- 不要修改 CONSTITUTION.md 或 RESEARCH.md
- 不要修改系统配置/网关
- 产出物必须落盘到指定路径
- 完成后输出简短摘要（3-5 行）

## 如果有 Review 回退意见
（Orchestrator 会在此处注入 REVIEW_REPORT.md 中的反馈，如有）
<review_feedback>
无
</review_feedback>
