---
cr_id: "CR-082"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "调整外部 process ledger 的项目命名空间、软链接目标、bootstrap 指针和 Git staged removals 语义，命中文件所有权、迁移和回退风险，必须使用 standard。"
rollback_to: "CR081 closed-current-delivery / /home/hyde/workspace/process as quant-lab-only ledger"
approval_result: "approved-cp8-closed-current-delivery"
created_at: "2026-06-17T09:11:43+08:00"
created_by: "host-orchestrator"
source: "user-request"
linked_issue: ""
parent_cr: "CR-081"
source_checkpoint: "current conversation"
source_decision_id: "USER-20260617-PROCESS-NAMESPACE-NORMALIZATION"
risk_class: "high"
owner: "host-orchestrator"
target_project_root: "/home/hyde/workspace/quant-lab"
external_process_root: "/home/hyde/workspace/process"
current_ledger_path: "/home/hyde/workspace/process"
target_ledger_path: "/home/hyde/workspace/process/quant-lab"
current_symlink: "/home/hyde/workspace/quant-lab/process -> ../process"
target_symlink: "/home/hyde/workspace/quant-lab/process -> ../process/quant-lab"
---

# CR-082 Process Ledger Project Namespace Normalization

## 变更描述

用户确认继续推进上一轮发现：当前 `/home/hyde/workspace/process` 是 quant-lab 专用 ledger，尚未按项目名分层。为了让外部 process 项目未来可以承载多个项目的过程文件，本 CR 拟将 quant-lab 过程账本从外部 process 根目录迁移到项目命名空间目录：

```text
/home/hyde/workspace/process/quant-lab
```

目标入口为：

```text
/home/hyde/workspace/quant-lab/process -> ../process/quant-lab
```

本 CR 只启动设计与执行门禁。CP2 / CP3 / CP5 approve 前不移动 `/home/hyde/workspace/process` 内容、不创建 `/home/hyde/workspace/process/quant-lab`、不替换 symlink、不修改 `LEDGER.md` / `ledger.yaml` / bootstrap 脚本、不执行 Git commit / push、不访问 NAS / data / reports / 凭据 / runtime。

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否改变外部 process 项目的组织模型 | `/home/hyde/workspace/process`、quant-lab ledger | true | 从单项目根目录调整为按项目名分层。 |
| 场景层 | 是否改变 bootstrap / clone 恢复路径 | `quant-lab/process` symlink、`scripts/link-engineering-ledger.sh` | true | 目标 symlink 改为 `../process/quant-lab`，脚本默认路径需更新。 |
| 计划层 | 是否需要门禁和回滚 | 外部目录、原 symlink、备份路径 | true | CP2/CP3/CP5 批准后执行 preflight -> namespace move -> symlink update -> verification。 |
| 安全层 | 是否引入误移动 / 误删风险 | 外部 ledger 根目录、`find -L`、`rsync --delete` | true | 禁止 delete、禁止跟随 symlink 批量操作，执行前保留回滚点。 |
| 交付层 | 是否影响 CR078 Git governance | staged `process/**` removals、pointer files | true | CR082 应先于 CR078，避免把错误目录模型提交到 Git。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-082-PROCESS-LEDGER-PROJECT-NAMESPACE-NORMALIZATION-2026-06-17.md` | 新增 | N/A | N/A | pending |
| `process/STATE.md` | 原文档更新 | 保留 CR081 closure 与 CR046 paused 主线 | `history` / `cr_tracking` / `human_gate_decisions` | pending |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留全部历史 CR 与候选 | `active_crs` | pending |
| `/home/hyde/workspace/process/README.md` | 后续执行时重写为总 process 项目 README | 当前 README 迁入 `quant-lab/README.md` | 执行证据 | pending |
| `LEDGER.md` / `ledger.yaml` | 后续执行时更新 | 保留 CR081 指针语义，切换到 `process/quant-lab` | 文件内容 | pending |
| `scripts/link-engineering-ledger.sh` | 后续执行时更新 | 保留安全检查，默认 target 改为 `../process/quant-lab` | 文件内容 | pending |
| `/home/hyde/workspace/quant-lab/process` | 后续执行时替换 symlink | 当前 symlink 可由 `ln -sfn ../process` 回退 | 执行证据 | pending |
| staged `process/**` removals | 不直接修改 | 保留 CR081 结果，CR078 再治理提交 / 推送 | CR078 | deferred |

## 冲突预检

| 对象 | 影响面 | 结论 | 说明 |
|---|---|---|---|
| `CR-081` | process externalization | dependency | CR081 已关闭，CR082 是其结构修正后续。 |
| `CR-078` | remote Git governance / publication | order-before | CR082 应先于 CR078，否则 remote governance 会固化当前错误命名空间。 |
| `CR-077` | old root retirement | no-overlap | 不删除旧根，不移动 `/home/hyde/workspace/local_backtest`。 |
| `CR-080` | data/reports restore | no-overlap | 不触碰 `data/` / `reports/` 内容。 |
| `CR-046` | QMT / MiniQMT paused runtime | no-overlap | 不恢复 CR046，不启动 runtime。 |

冲突预检结论：允许创建正式 CR082 并发起 CP2/CP3/CP5 人工门禁；批准前不得移动外部 process 内容或替换 symlink。

## 推荐方案

推荐将外部 process 项目变为总容器，quant-lab 作为项目命名空间目录：

```text
/home/hyde/workspace/
  quant-lab/
    process -> ../process/quant-lab

  process/
    README.md
    quant-lab/
      README.md
      STATE.md
      changes/
      checks/
      checkpoints/
      context/
      ...
```

执行策略为：先在同一 filesystem 内创建 staging / backup，再把现有 ledger 内容移动到 `quant-lab/`，在 root 写总 README，最后替换 symlink 并验证 `process/STATE.md`、`scripts/link-engineering-ledger.sh` 和 CR tracking。

## 分阶段执行设计

| 阶段 | 动作 | 批准前是否允许 | 说明 |
|---|---|---:|---|
| Gate | 创建 CR082、context、CP2/CP3/CP5 checks/checkpoints、launch message | done | 本轮只发起门禁。 |
| Preflight | 检查 `/home/hyde/workspace/process` 为真实目录、`quant-lab/` 不存在或为空、symlink 当前可读、staged removals 数量 | no | 需 CP2/CP3/CP5 approve。 |
| Namespace Move | 移动外部 ledger 内容到 `/home/hyde/workspace/process/quant-lab/` | no | 禁止 delete，禁止跨 filesystem rename 不可验证路径。 |
| Root README | 写 `/home/hyde/workspace/process/README.md` 为总 process 项目说明 | no | 当前 Quant Lab README 移入 namespace。 |
| Pointer Update | 更新 `LEDGER.md`、`ledger.yaml`、bootstrap 脚本和 symlink | no | 默认 target `../process/quant-lab`。 |
| Verification | 验证 `process/STATE.md`、CR tracking、human gate、symlink 和 diff check | no | 不触碰 remote、data、reports、凭据或 runtime。 |
| CP8 | 发布就绪、回滚说明、风险接受 | pending | 执行后发起。 |

## 不授权项

| ID | 不授权内容 |
|---|---|
| NA-CR082-001 | CP2/CP3/CP5 approved 前，不移动 `/home/hyde/workspace/process` 内容，不创建或替换 `/home/hyde/workspace/process/quant-lab`。 |
| NA-CR082-002 | 不执行 Git commit / push / remote add / remote set-url / branch governance / tag / history rewrite / force push。 |
| NA-CR082-003 | 不读取或修改 `data/`、`reports/` 内容，不做 checksum、样本解析、copy、restore、delete 或 overwrite。 |
| NA-CR082-004 | 不读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。 |
| NA-CR082-005 | 不访问 NAS 内容、不创建 NAS mount、不写 NAS、不删除 NAS 文件、不做 NAS promote。 |
| NA-CR082-006 | 不执行 provider fetch、lake write、catalog publish 或 MARKET_DATA_LAKE_ROOT replacement。 |
| NA-CR082-007 | 不启动 QMT / MiniQMT runtime，不恢复 CR046，不执行交易相关操作。 |
| NA-CR082-008 | 不删除外部 process ledger、CR081 backup、旧根或 target root。 |
| NA-CR082-009 | 不执行 `rsync --delete`、`find -L` 或跟随 symlink 的 destructive cleanup。 |

## 推荐门禁

| Gate | 文件 | 当前状态 | 说明 |
|---|---|---|---|
| CP2 | `process/checkpoints/CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE.md` | approved | 用户于 2026-06-17T09:32:20+08:00 回复“同意”，确认按项目名分层的范围和前置顺序。 |
| CP3 | `process/checkpoints/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-REVIEW.md` | approved | 用户于 2026-06-17T09:32:20+08:00 回复“同意”，确认目录模型、symlink 和 bootstrap 架构。 |
| CP5 | `process/checkpoints/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS.md` | approved | 用户于 2026-06-17T09:32:20+08:00 回复“同意”，确认执行顺序、回滚点和实际授权边界。 |
| CP8 | `process/checkpoints/CP8-CR082-DELIVERY-READINESS.md` | approved | 用户于 2026-06-17T09:57:53+08:00 回复“同意，下一步做什么。你需要准备足够的上下文。我需要清除上下文了”，确认 READY_WITH_RISK 和 R-CR082-001..03。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险文档更新 | false | 会移动外部 ledger namespace 并替换 symlink。 |
| 修改权限 / 安全边界 | true | 涉及软链接、外部目录 owner 和 destructive 操作边界。 |
| 修改安装 / bootstrap 路径 | true | 新环境恢复路径从 `/process` 变为 `/process/quant-lab`。 |
| 修改外部接口或文件 owner | true | 外部 process 根目录从项目 ledger 变为总容器。 |
| 是否保持 fast-lane | false | standard。 |

## LLD 设计批次门禁

- 是否需要 Story LLD 设计批次：false
- 替代门禁：`CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS`
- 理由：CR082 是 repository/process-ledger namespace migration gate，不修改应用业务代码、API、数据 schema 或策略逻辑；执行设计由 CP3/CP5 Decision Brief 承载。

## 处理结论

- 当前结论：`closed-current-delivery / READY_WITH_RISK`
- 当前门禁：CP8 已人工确认。
- 下一步：建议启动 CR078 remote Git governance / publication gate，处理 staged `process/**` removals、pointer files、CR081/CR082 quality/release docs 的提交/推送策略；启动前仍不授权 Git commit/push/remote write。
