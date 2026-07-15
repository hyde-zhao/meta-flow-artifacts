---
story_id: STORY-024-03
story_name: policy-route-execution skill + op_mapper
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-024-ptm-te-agent
tier: M
wave: W2
depends_on: [STORY-024-01]
blocks: [STORY-024-04]
status: lld-ready
lld_policy: full-lld
feature_design_refs: [ptm-te-agent]
file_ownership:
  - skills/policy-route-execution/SKILL.md
  - skills/policy-route-execution/scripts/op_mapper.py
parallel_safe: true
hld_ref: process/HLD-CR-024.md
lld_ref: process/stories/STORY-024-03-policy-route-execution-LLD.md
---

# STORY-024-03：policy-route-execution skill + op_mapper

## 1. 目标

新建 policy-route-execution skill（SKILL.md 按 ptm-atomic 真实 CLI 重写 + op_mapper.py 双层映射必需脚本）。op_mapper.py 承载 op_id->子命令（8 个）+ args->flag（7 op）双层映射，是 CR-024 核心复杂点。

## 2. 范围

- **新建文件**：2 个（SKILL.md + scripts/op_mapper.py）
- **真相源**：ptm-atomic 仓库 run_policy_route.py + op yaml + `ptm-atomic run --help` + `ptm-atomic list`
- **不改动**：agents/ptm-te.md（S1）、device skill（S2）、install.py（S4）
- **详细设计证据**：`process/stories/STORY-024-03-policy-route-execution-LLD.md`（full-lld 14 章）

## 3. 验收标准

- [ ] AC01：op_mapper.py 覆盖 8 个 op_id->子命令映射（含 login）
- [ ] AC02：op_mapper.py 覆盖 7 个 op 的 args->flag 映射（config/update/delete/verify/priority/reset-hitcount/verify-hitcount）
- [ ] AC03：映射表与 `run_policy_route.py` + op yaml `inputs.params` + `ptm-atomic run --help` 三处真相源一致性校验 PASS
- [ ] AC04：rollback 处理与 `ptm-atomic list` 实测一致（config->inverse_op:delete；update->restore_snapshot；delete 清理动作不回滚；priority 无元数据；reset-hitcount irreversible 不回滚；verify 只读）
- [ ] AC05：干跑/执行/verify 三阶段（默认 --dry-run，--execute 需授权）
- [ ] AC06：STATE_INVALID 自动重连（重新 auth login + 重试 1 次）
- [ ] AC07：inverse_op 清理（config 后执行 delete 清理）
- [ ] AC08：SKILL.md 含错误表（含 ePolicyRouteInIfModeError）+ Gotchas（CLI 名/扁平格式/三层命名/rollback 字段/reset-hitcount/session/login --password-env/update --id/接口模式）
- [ ] AC09：未识别 op_id 阻塞并提示工具缺失（反馈 ptm-tae）
- [ ] AC10：前置 `ptm-atomic sync` 验证通过

## 4. 依赖与门控

- **依赖**：STORY-024-01（agent 骨架定义三层映射契约 + 执行流程 [4]）
- **阻塞**：STORY-024-04（安装器需注册 policy-route-execution skill）
- **并行**：与 STORY-024-02 文件所有权无冲突，W2 可并行
- **dev_gate**：S3 LLD 确认后进入实现
