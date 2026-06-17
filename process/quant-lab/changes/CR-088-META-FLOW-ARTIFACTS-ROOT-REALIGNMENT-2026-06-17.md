---
cr_id: "CR-088"
status: "closed-local-remediation"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "纠正过程归档 artifact root 与软链接目标，涉及文件所有权与路由元数据。"
rollback_to: "CR087 route baseline under /home/hyde/workspace/process/quant-lab"
approval_result: "completed"
created_at: "2026-06-17T19:50:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-17T19:50:00+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-087"
source_checkpoint: "current conversation"
source_decision_id: "USER-20260617-META-FLOW-ARTIFACTS-ROOT-CORRECTION"
follow_up_type: "artifact-root-realignment"
risk_class: "medium"
owner: "host-orchestrator"
revisit_condition: "process or docs symlinks point outside /home/hyde/workspace/meta-flow-artifacts/process/quant-lab"
acceptance_criteria: "process/docs/checkpoints links point to /home/hyde/workspace/meta-flow-artifacts/process/quant-lab and route checks pass"
close_condition: "new artifact root is readable; CR tracking consistency passes; no forbidden external action executed"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR088 Meta Flow Artifacts Root Realignment

## 变更描述

用户指出 meta-flow 过程文件归档目录应使用 `/home/hyde/workspace/meta-flow-artifacts`，而不是 `/home/hyde/workspace/process`。本 CR 纠正 CR087 的 artifact root。

正确目标遵循 `/home/hyde/workspace/meta-flow-artifacts/README.md`：

```text
process/<project-name>/
```

因此 quant-lab 的过程根应为：

```text
/home/hyde/workspace/meta-flow-artifacts/process/quant-lab
```

## 为什么上一轮链接到了错误位置

上一轮依据当前仓库 `AGENTS.md`、`LEDGER.md`、`ledger.yaml` 和 `process/STATE.md` 中 CR081 / CR082 的既有状态执行；这些文件当时都指向 `/home/hyde/workspace/process/quant-lab`。用户当前纠正了最新归档根，因此本 CR 以当前用户指令和 `meta-flow-artifacts/README.md` 为准。

## 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 | 批准状态 |
|---|---|---|---|
| `process/STATE.md` | 原文档更新 | 保留 CR081/CR082/CR087 历史，追加 CR088 current route | approved |
| `process/.meta-flow-process.yaml` | 原文档更新 | 保留旧路由为历史，current route 改为 meta-flow-artifacts | approved |
| `LEDGER.md` | 原文档更新 | 保留说明并切换路径 | approved |
| `ledger.yaml` | 原文档更新 | 保留说明并切换路径 | approved |
| `scripts/link-engineering-ledger.sh` | 原文档更新 | 默认 root 改为 meta-flow-artifacts，可用 PROCESS_LEDGER_ROOT override | approved |
| `process` / `docs/*` / `checkpoints` symlink | 原文档更新 | 旧 `/home/hyde/workspace/process/quant-lab` 不删除，仅作为 legacy copy | approved |

## 五维度影响分析

| 维度 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|
| 需求层 | 业务需求 | false | 不修改 |
| 场景层 | 测试 / 验证场景 | false | 不修改 |
| 计划层 | process routing | true | 复制 ledger 并重建软链 |
| 安全层 | 本地文件系统路径 | true | 不删除旧目录，不读 data/reports，不触碰凭据 |
| 交付层 | pointer metadata / bootstrap | true | 更新 `.gitignore`、`LEDGER.md`、`ledger.yaml`、script 和 route metadata |

## 回退决策

- 回退到：`/home/hyde/workspace/process/quant-lab`
- 回退方式：将 symlink 重新指回 CR087 目标；不删除新旧任一归档目录。

## 禁止动作

- 不删除 `/home/hyde/workspace/process/quant-lab`
- 不执行远端 Git 写
- 不读取 `.env` / 凭据
- 不读取或迁移 data/reports 内容
- 不执行 NAS 内容操作、provider/lake/catalog、runtime 或 cleanup

## 路由验证结果

| 检查项 | 结果 | 证据 |
|---|---|---|
| 正确 artifact root | PASS | `/home/hyde/workspace/meta-flow-artifacts` |
| 正确 process root | PASS | `/home/hyde/workspace/meta-flow-artifacts/process/quant-lab` |
| `process` 软链 | PASS | `process -> ../meta-flow-artifacts/process/quant-lab` |
| `docs/design` 软链 | PASS | `../../meta-flow-artifacts/process/quant-lab/docs/design` |
| `docs/features` 软链 | PASS | `../../meta-flow-artifacts/process/quant-lab/docs/features` |
| `docs/quality` 软链 | PASS | `../../meta-flow-artifacts/process/quant-lab/docs/quality` |
| `checkpoints` 软链 | PASS | `../meta-flow-artifacts/process/quant-lab/checkpoints` |
| 内部 release 软链 | PASS | 指向 `../../../meta-flow-artifacts/process/quant-lab/docs/release/*` |
| 遗留副本 | PASS | `/home/hyde/workspace/process/quant-lab` 保留，未删除 |

## 处理结论

CR088 已关闭为 `closed-local-remediation`。当前路由以 `/home/hyde/workspace/meta-flow-artifacts/process/quant-lab` 为准。
