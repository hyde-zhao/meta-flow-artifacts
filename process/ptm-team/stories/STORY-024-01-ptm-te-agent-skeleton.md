---
story_id: STORY-024-01
story_name: ptm-te agent 骨架与编排流程
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-024-ptm-te-agent
tier: S
wave: W1
depends_on: []
blocks: [STORY-024-02, STORY-024-03, STORY-024-04]
status: lld-ready
lld_policy: full-lld
feature_design_refs: [ptm-te-agent]
file_ownership:
  - agents/ptm-te.md
parallel_safe: false
hld_ref: process/HLD-CR-024.md
lld_ref: process/stories/STORY-024-01-ptm-te-agent-skeleton-LLD.md
---

# STORY-024-01：ptm-te agent 骨架与编排流程

## 1. 目标

重写 `agents/ptm-te.md`（50 行 planned 占位 -> active 编排器），定义 ptm-te 编排流程、PC 消费契约、三层映射声明、login-once-reuse、inverse_op 清理、运行时目录、执行门控。是 S2/S3/S4 的基础（定义全部下游 skill 消费契约）。

## 2. 范围

- **修改文件**：`agents/ptm-te.md`（唯一文件，frontmatter 升格 status: active, color: green）
- **不改动**：ptm-tde 已交付基线、install.py（S4）、skill 文件（S2/S3）
- **详细设计证据**：`process/stories/STORY-024-01-ptm-te-agent-skeleton-LLD.md`（full-lld 14 章）

## 3. 验收标准

- [ ] AC01：`agents/ptm-te.md` frontmatter `status: active`，`color: green`
- [ ] AC02：编排流程 [1]-[8] 完整（用例解析 cases/upload/ -> 设备准备 -> login -> 逐条 op 执行 -> 结果判定 -> 执行日志 -> 用例清理 inverse_op -> 快照 after + 回写）
- [ ] AC03：PC 消费契约声明（`case_steps[].step_name` / `atomic_op.op_id` / `atomic_op.args` / `expected_result`，输入 `cases/upload/`）
- [ ] AC04：三层映射声明（8 op_id->子命令 + 7 op args->flag，引用 HLD §4，op_mapper 在 S3 实现）
- [ ] AC05：login-once-reuse-session 声明（auth login 一次，session.json 复用，STATE_INVALID 重连）
- [ ] AC06：inverse_op 清理声明（config->delete；update->restore_snapshot；irreversible 不回滚，引用 HLD §9）
- [ ] AC07：运行时目录声明（`cases/upload/` + `runs/<run-id>/` 含 `report.md`）
- [ ] AC08：执行门控写进 agent md（不复用 checkpoint-manager）
- [ ] AC09：dry-run 默认门声明（默认 --dry-run，--execute 单次授权，CP2 DQ-01）

## 4. 依赖与门控

- **依赖**：无（W1 基础）
- **阻塞**：STORY-024-02 / 03 / 04（下游 skill 与安装器需 agent 骨架定义的契约）
- **dev_gate**：S1 LLD 确认后进入实现
