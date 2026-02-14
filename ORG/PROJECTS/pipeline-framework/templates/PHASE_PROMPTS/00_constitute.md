# Phase 0: Constitute（立宪）

你正在执行项目 `<project>` 的 Pipeline Phase 0: Constitute。

## 目的
定义项目的基本原则、技术约束、质量标准。这是整个 Pipeline 的基石，后续所有阶段都会参照此文件。

## 你需要读的文件
- `ORG/PROJECTS/<project>/STATUS.md`（项目当前状态）
- `ORG/PROJECTS/<project>/DECISIONS.md`（已有决策）
- `ORG/DEPARTMENTS/<dept>/CHARTER.md`（部门职责与边界）
- 项目 repo 中的已有文档（如 README、REPORT 等）

## 你需要产出的文件
写入路径：`ORG/PROJECTS/<project>/pipeline/CONSTITUTION.md`

必须包含以下章节：
1. **项目目标**（1-3 句话，明确要达成什么）
2. **技术栈约束**（语言、框架、依赖限制、运行环境）
3. **质量标准**（测试覆盖率要求、性能指标、文档要求）
4. **边界约束**（不做什么、安全红线、资源限制）
5. **与 ORG 章程的对齐声明**（确认遵守 Boot/Closeout/落盘/变更控制）

## 完成标准
- CONSTITUTION.md 存在且非空
- 包含以上 5 个章节
- 每个章节至少有 2 条具体条目

## 约束
- 不要修改系统配置/网关
- 产出物必须落盘到指定路径
- 完成后输出简短摘要（3-5 行）
