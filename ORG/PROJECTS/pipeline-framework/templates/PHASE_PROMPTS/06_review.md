# Phase 6: Review（审查）

你正在执行项目 `<project>` 的 Pipeline Phase 6: Review。

## 目的
对整个 Pipeline 本轮的产出进行质量审查和目标达成分析。你的判定（PASS/FAIL）决定 Pipeline 是归档完成还是回退重做。

## 你需要读的文件（全部）
- `ORG/PROJECTS/<project>/pipeline/CONSTITUTION.md`（项目原则与质量标准）
- `ORG/PROJECTS/<project>/pipeline/RESEARCH.md`（调研报告）
- `ORG/PROJECTS/<project>/pipeline/SPECIFICATION.md`（需求规格与验收标准）
- `ORG/PROJECTS/<project>/pipeline/PLAN.md`（实现计划）
- `ORG/PROJECTS/<project>/pipeline/TASKS.md`（任务列表）
- `ORG/PROJECTS/<project>/pipeline/IMPL_STATUS.md`（实现状态）
- `ORG/PROJECTS/<project>/pipeline/TEST_REPORT.md`（测试报告）
- 项目 repo 中的代码文件（抽查关键模块）

## 审查维度

### 1. 规格符合度（权重 30%）
- 代码是否满足 SPECIFICATION.md 的所有功能需求？
- 非功能需求是否满足？
- 接口定义是否一致？

### 2. 质量标准（权重 25%）
- 是否满足 CONSTITUTION.md 定义的质量标准？
- 代码风格、命名、结构是否合理？
- 是否有明显的 bug、安全漏洞、性能问题？

### 3. 测试充分性（权重 20%）
- TEST_REPORT.md 是否覆盖了所有关键路径？
- 验收测试通过率是否达标？
- 失败用例的根因分析是否合理？

### 4. 可维护性（权重 15%）
- 代码注释和文档是否足够？
- 模块划分是否清晰？
- 后续迭代是否容易扩展？

### 5. 目标达成（权重 10%）
- 整体是否达到了 CONSTITUTION.md 定义的项目目标？
- 是否有遗漏的关键功能？

## 你需要产出的文件
写入路径：`ORG/PROJECTS/<project>/pipeline/REVIEW_REPORT.md`

必须包含以下章节：
1. **审查摘要**（一段话总结）
2. **各维度评分**
   ```
   规格符合度:  X/5
   质量标准:    X/5
   测试充分性:  X/5
   可维护性:    X/5
   目标达成:    X/5
   加权总分:    X.X/5.0
   ```
3. **总体判定**: `PASS` 或 `FAIL`
   - PASS 条件：加权总分 >= 3.5 且无任何维度 <= 2
   - 否则 FAIL
4. **如果 PASS**：
   - 本轮亮点（2-3 条）
   - 下一轮建议改进方向
5. **如果 FAIL**：
   - 具体问题列表（每条附所在文件/行号）
   - 建议回退到哪个阶段（constitute/research/specify/plan/implement/test）
   - 回退理由
   - 给回退阶段的具体修改建议

## 完成标准
- REVIEW_REPORT.md 存在且非空
- 包含 5 个维度的评分
- 包含明确的 PASS/FAIL 判定
- 如果 FAIL，包含回退目标和具体修改建议

## 约束
- 不要修改任何 pipeline 文件或代码文件（你是审查者，不是修改者）
- 不要修改系统配置/网关
- 保持客观公正，不要因为"差一点就通过"而放水
- 完成后输出简短摘要（3-5 行）
