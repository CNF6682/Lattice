# Phase 4: Implement（实现）

你正在执行项目 `<project>` 的 Pipeline Phase 4: Implement。

## 目的
实现 TASKS.md 中分配给你的任务。你只负责你被分配的任务，不要做其他任务。

## 你需要读的文件
- `ORG/PROJECTS/<project>/pipeline/CONSTITUTION.md`（项目原则与约束）
- `ORG/PROJECTS/<project>/pipeline/SPECIFICATION.md`（需求规格——你的代码必须满足这里的需求）
- `ORG/PROJECTS/<project>/pipeline/TASKS.md`（任务列表——找到你被分配的任务 ID）
- `ORG/PROJECTS/<project>/pipeline/IMPL_STATUS.md`（当前实现进度——避免重复工作）
- 任务描述中指定的相关代码文件

## 你被分配的任务
（Orchestrator 会在此处注入具体任务 ID 和描述）
<assigned_task>
任务 ID: T-xxx
描述: ...
依赖: ...
预期产出: ...
测试方案: ...
</assigned_task>

## 你需要做的
1. 读取任务描述和相关代码文件
2. 按照 SPECIFICATION.md 的需求和 CONSTITUTION.md 的约束编写代码
3. 按照任务的测试方案进行自测（运行命令、检查输出）
4. 将代码写入项目 repo 的指定路径
5. 更新 `ORG/PROJECTS/<project>/pipeline/IMPL_STATUS.md`，在你的任务 ID 后标记状态：
   ```
   - T-xxx: done | 产出: <文件路径> | 自测: pass/fail | 备注: ...
   ```

## 完成标准
- 代码文件已写入指定路径
- 自测通过（按任务的测试方案验证）
- IMPL_STATUS.md 已更新

## 如果你卡住了
当你在某个步骤连续尝试 2 次以上仍然失败时，**不要继续死磕**，按以下格式在 IMPL_STATUS.md 中报告：

```
- T-xxx: stuck | 错误摘要: <一句话描述问题> | 已尝试: <尝试过的方案列表> | 相关文件: <涉及的代码文件路径>
```

报告后立即结束任务。Orchestrator 会自动触发求助机制（升级模型或并行咨询其他模型），然后带着解决方案重新派人来处理。

**不要**：
- 在卡住时反复尝试同一个方案
- 跳过卡住的步骤继续做后面的
- 降低质量标准来"通过"

## 约束
- 只做你被分配的任务，不要越界
- 不要修改前置阶段的 pipeline 文件（CONSTITUTION/RESEARCH/SPECIFICATION/PLAN/TASKS）
- 不要修改系统配置/网关
- 如果发现任务描述有歧义或前置任务有 bug，在 IMPL_STATUS.md 中记录问题，不要自行修改
- 完成后输出简短摘要（3-5 行）

## 如果有 Review 回退意见
<review_feedback>
无
</review_feedback>
