# Phase 5: Test（测试）

你正在执行项目 `<project>` 的 Pipeline Phase 5: Test。

## 目的
对 Phase 4 的实现进行分级测试，生成测试报告。测试的权威标准是 SPECIFICATION.md 中的验收条件。

## 你需要读的文件
- `ORG/PROJECTS/<project>/pipeline/CONSTITUTION.md`（质量标准）
- `ORG/PROJECTS/<project>/pipeline/SPECIFICATION.md`（验收标准——逐条验证）
- `ORG/PROJECTS/<project>/pipeline/TASKS.md`（每个任务的测试方案）
- `ORG/PROJECTS/<project>/pipeline/IMPL_STATUS.md`（实现状态——哪些任务 done、产出路径）
- 项目 repo 中的代码文件

## 测试分级

### Level 1: 单元测试
- 逐任务执行 TASKS.md 中定义的测试方案
- 记录每个任务的 pass/fail

### Level 2: 集成测试
- 测试跨任务的接口和数据流
- 验证模块间的交互是否正确

### Level 3: 验收测试
- 逐条对照 SPECIFICATION.md 的验收标准
- 对每个 FR-xxx 的验收条件执行验证
- 记录 pass/fail/skip（skip 需说明原因）

## 你需要产出的文件
写入路径：`ORG/PROJECTS/<project>/pipeline/TEST_REPORT.md`

必须包含以下章节：
1. **测试执行摘要**
   - 总数 / 通过 / 失败 / 跳过
   - 按级别统计（单元 / 集成 / 验收）
2. **单元测试详情**
   - 每个任务 ID + 测试命令 + 结果 + 输出摘要
3. **集成测试详情**
   - 测试场景 + 结果 + 问题描述（如有）
4. **验收测试详情**
   - 每个 FR-xxx + 验收条件 + pass/fail/skip + 证据（命令输出或截图路径）
5. **失败分析**
   - 每个失败用例的根因分析和修复建议
6. **测试覆盖率**（如适用）
   - 代码覆盖率数据或估算

## 完成标准
- TEST_REPORT.md 存在且非空
- 所有 TASKS.md 中的任务都有单元测试结果
- 所有 SPECIFICATION.md 中的 FR-xxx 都有验收测试结果
- 验收测试通过率 >= 阈值（默认 80%，以 CONSTITUTION.md 中的质量标准为准）

## 如果你卡住了
当测试环境出现无法解决的问题（依赖安装失败、环境不兼容等），或某个测试反复失败且你无法判断是代码 bug 还是测试方案问题时，**不要继续死磕**，在 TEST_REPORT.md 中标记：

```
### Stuck Items
- 测试项: <FR-xxx 或 T-xxx> | 错误摘要: <一句话> | 已尝试: <方案列表> | 相关文件: <路径>
```

报告后立即结束任务。Orchestrator 会自动触发求助机制。

## 约束
- 不要修改代码文件（测试阶段只读代码、只运行测试）
- 不要修改前置阶段的 pipeline 文件
- 不要修改系统配置/网关
- 如果测试环境有问题导致无法执行某些测试，标记为 skip 并说明原因
- 完成后输出简短摘要（3-5 行）
