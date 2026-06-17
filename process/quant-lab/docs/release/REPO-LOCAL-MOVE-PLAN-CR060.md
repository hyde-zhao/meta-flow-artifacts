---
status: "pending-cp5-approval"
version: "1.0"
change_id: "CR-060"
title: "Repo-local Mechanical Migration Execution Plan"
created_by: "host-orchestrator"
created_at: "2026-06-15T06:43:41+08:00"
execute_allowed_now: false
---

# CR060 Repo-local Move Plan

## 1. 执行定位

CR060 的第一切片不是目录搬家，而是 repo-local future-facing 文档身份收敛。原因是当前仓库名、工作目录、包名、import path、Git remote 和数据湖 root 仍有独立风险，不能在一个 CR 中混合执行。

| 层级 | 本 CR 处理 | 后续处理 |
|---|---|---|
| README / USER-MANUAL 项目身份 | 是，CP5 approve 后执行 | N/A |
| 历史过程证据 | 否，preserve-audit | 单项豁免需新决策 |
| 包名 / import path | 否 | CR061 |
| Git remote / branch / push | 否 | CR062 |
| NAS archive / research data | 否 | CR063 |
| Data lake root / catalog | 否 | CR064 |
| QMT / MiniQMT runtime | 否 | CR065 或恢复 CR046 |

## 2. 真实执行清单

| Action ID | 文件 | 动作 | 前置条件 | 验证 | 回滚 |
|---|---|---|---|---|---|
| CR060-ACT-001 | `README.md` | 文本改写：`quant-lab` canonical + `local_backtest` legacy alias | CP5 `approve` | `rg -n "local_backtest\|quant-lab" README.md` 人工复核 | Git diff 反向补丁 / CR059 bundle |
| CR060-ACT-002 | `docs/USER-MANUAL.md` | 文本改写：用户手册入口同步身份 | CP5 `approve` | `rg -n "local_backtest\|quant-lab" docs/USER-MANUAL.md` 人工复核 | Git diff 反向补丁 / CR059 bundle |

## 3. 明确不执行的动作

| 操作 | 原因 |
|---|---|
| `mv` / `rename` / `rm` | 本切片不需要文件系统移动；若后续需要目录重排，进入 CR061。 |
| repo-wide bulk replace | 会破坏历史审计和运行配置语义。 |
| `pyproject.toml` package rename | 需要 import compatibility、测试矩阵和发布策略。 |
| `MARKET_DATA_LAKE_ROOT` 改名或 root 切换 | 数据湖入口必须独立备份和 publish gate。 |
| `git remote add` / `git push` | Git release identity 属于 CR062。 |
| NAS 数据迁移 | NAS 目录已准备，但真实搬家属于 CR063。 |

## 4. 执行窗口规则

| 规则 ID | 规则 | 失败行为 |
|---|---|---|
| MW-CR060-001 | CP5 未 `approve` 前不得改写 README / USER-MANUAL。 | BLOCKED |
| MW-CR060-002 | 如果 HEAD 变化，先刷新 manifest 中 source_commit 和 rollback evidence。 | BLOCKED |
| MW-CR060-003 | 如果发现未纳入 manifest 的用户改动影响同一段落，先生成 include/exclude 说明。 | BLOCKED |
| MW-CR060-004 | 任何需要读取 `.env` 或凭据正文的判断一律停止。 | BLOCKED |

## 5. 验证策略

| 验证项 | 命令 / 方法 | 预期 |
|---|---|---|
| YAML 解析 | 解析 `docs/release/MIGRATION-EXECUTION-MANIFEST-CR060.yaml` | PASS |
| 人工门禁格式 | `scripts/check_human_gate_decision_brief.py` | PASS |
| CR tracking | `scripts/check_cr_tracking_consistency.py --project-root .` | PASS |
| Diff whitespace | `git diff --check` | PASS |
| 文档引用复核 | `rg` 查看 touched docs 中 `local_backtest` / `quant-lab` | 剩余 `local_backtest` 均为 legacy / historical / runtime context |

## 6. 回滚策略

| 回滚对象 | 回滚方式 |
|---|---|
| README / USER-MANUAL 文本改写 | 使用执行后的 Git diff 反向补丁，或从 CR059 cold backup / Git bundle 恢复。 |
| CR060 门禁文档 | 后续 CR 修订，不使用 destructive git reset。 |
| 后续错误扩大到未授权范围 | 立即停止，记录 failure manifest，交回人工决策。 |
