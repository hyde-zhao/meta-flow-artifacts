---
cr_id: "CR-071"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中本机项目根迁移、Git 工作树影子根、排除敏感/大体积目录、回滚与验证门禁；不得使用 fast-lane。"
rollback_to: "pre-CR071 local_backtest shadow-copy baseline"
approval_result: "cp8-approved-closed-current-delivery"
created_at: "2026-06-16T07:16:44+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-16T07:33:48+08:00"
cp8_approved_by: "user"
cp8_approved_at: "2026-06-16T09:13:26+08:00"
closed_at: "2026-06-16T09:13:26+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-070"
source_checkpoint: "process/checkpoints/CP8-CR070-DELIVERY-READINESS.md"
source_decision_id: "USER-20260616-START-CR071-MIG-A"
follow_up_type: "copy-first-shadow-root-migration"
risk_class: "local_filesystem_shadow_root_migration"
owner: "host-orchestrator"
revisit_condition: "CP2/CP3/CP5 修改、目标目录已存在且非空、rsync 不可用、排除清单变化、需要复制 .env / data / reports / NAS / Git remote / data lake / QMT runtime 时重访。"
acceptance_criteria: "在 CP2/CP3/CP5 approved 后，将 /home/hyde/workspace/local_backtest copy-first 到同级 /home/hyde/workspace/quant-lab；旧目录保留；不复制 reports/、data/、.venv/、node_modules/；默认不复制 .env；不执行 NAS、远端 Git 写动作、QMT runtime 或数据湖切换。"
close_condition: "closed at 2026-06-16T09:13:26+08:00 after CP8 READY_WITH_RISK approved by user; copy-first 执行完成，本地静态验证通过，reports Git 风险已在 CP8 前缓解，root cutover / runtime / data lake / remote Git 仍后移且未授权。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-071 Copy-first Shadow Root Migration (MIG-A)

## 变更描述

用户要求启动 `CR071 / MIG-A`：按 copy-first shadow root 方案，将当前工作区 `/home/hyde/workspace/local_backtest` 复制迁移到同级 `/home/hyde/workspace/quant-lab`。旧目录必须保留，不删除、不重命名；本轮不复制 `reports/`、`data/`、`.venv/`、`node_modules/`，`.env` 默认不复制；只有 CR071 的 CP2 / CP3 / CP5 门禁通过后，才允许执行本机 copy-first 迁移。

本 CR 明确不是 NAS 迁移、不是远端 Git 发布、不是 QMT / MiniQMT runtime 恢复、不是数据湖 root 切换，也不是历史过程证据批量改写。

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 当前 active formal CR | `CR-046` paused | CR046 保持用户暂停的 CP6 PASS / ready-for-verification；CR071 不恢复 CR046。 |
| 与 CR046 交易/runtime 影响面重叠 | false | CR071 不触碰 QMT / MiniQMT runtime、账户、凭据、订单、策略导入或交易动作。 |
| 与 CR053-CR070 迁移治理关系 | consumer / continuation | CR071 消费 CR053 inventory、CR060 identity、CR061 package/import、CR066-CR070 repository hygiene / governance 结果。 |
| 目标目录状态 | clear | `/home/hyde/workspace/quant-lab` 当前不存在；若执行前变为非空，CP5 后置 preflight 必须阻断并重新确认。 |
| 本轮授权范围 | local copy only | 仅允许本机 copy-first；不允许 NAS、远端 Git write、data lake 切换、runtime 或凭据读取。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-071-COPY-FIRST-SHADOW-ROOT-MIGRATION-2026-06-16.md` | 新增 | N/A | 本文件 | approved |
| `docs/release/COPY-FIRST-SHADOW-ROOT-MIGRATION-PLAN-CR071.md` | 新增 | N/A | 文档头部 / 本 CR | approved |
| `docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml` | 新增 | N/A | manifest | approved |
| `process/context/CP2-CR071-REQUIREMENT-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/context/CP3-CR071-DESIGN-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/context/CP5-CR071-EXECUTION-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/context/CP8-CR071-DELIVERY-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/checks/CP2-CR071-REQUIREMENTS-BASELINE.md` | 新增 | N/A | 自动预检 | approved |
| `process/checkpoints/CP2-CR071-REQUIREMENTS-BASELINE.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checks/CP3-CR071-HLD-CONSISTENCY.md` | 新增 | N/A | 自动预检 | approved |
| `process/checkpoints/CP3-CR071-HLD-REVIEW.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checks/CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS.md` | 新增 | N/A | 自动预检 | approved |
| `process/checkpoints/CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checks/CP8-CR071-DELIVERY-READINESS.md` | 新增 | N/A | 自动预检 | approved |
| `process/checkpoints/CP8-CR071-DELIVERY-READINESS.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/release/RELEASE-CONTEXT-CR071.yaml` | 新增 | N/A | release context | approved |
| `docs/release/RELEASE-NOTES-CR071.md` | 新增 | N/A | release notes | approved |
| `docs/release/DEPLOY-CHECKLIST-CR071.md` | 新增 | N/A | deploy checklist | approved |
| `docs/release/ROLLBACK-CR071.md` | 新增 | N/A | rollback | approved |
| `docs/release/MIGRATION-CR071.md` | 新增 | N/A | migration | approved |
| `docs/release/FEEDBACK-CR071.md` | 新增 | N/A | feedback | approved |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused 与 CR070/CR072 closed 事实；将 CR071 移入 closed | `history` / `cr_tracking` | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR046 paused 与 closed CR 列表；将 CR071 移入 closed | `closed_crs` | approved |
| `README.md` / `docs/USER-MANUAL.md` | 不变 | CR060 已完成 identity 收敛；本 CR 不改文档正文 | 不适用 | n/a |
| `pyproject.toml` / `uv.lock` / `quant_lab/**` | 不变 | CR061 已完成 package/import convergence；本 CR 不改依赖或包布局 | 不适用 | n/a |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| `/home/hyde/workspace/local_backtest` | `/home/hyde/workspace/quant-lab` | 旧目录原样保留 | 新目录作为 shadow root；不删除、不改名旧目录。 |
| `local_backtest` legacy alias | `quant-lab` canonical root | 原文保留 | 历史 `process/**` 证据继续保留当时事实；本 CR 不批量改写历史引用。 |
| CR053 inventory / dry-run | CR071 copy-first execution plan | 原文保留 | CR071 只执行本机目录复制，不继承 NAS / data lake / runtime / remote Git 授权。 |
| CR060/CR061 identity + package convergence | CR071 target shadow root | 原文保留 | 复制后新根保留当前文件内容和 Git 工作树状态，后续如需根路径切换另行确认。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | MIG-A 本机 copy-first 范围 | true | 新增 CR071 CP2 scope baseline；不修改全局产品需求正文。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | copy-first、排除清单、目标根验证 | true | CP5 定义本地静态验证：目录存在、排除项不存在、Git 状态可读。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR071 CP2/CP3/CP5 -> copy execution -> CP6/CP7/CP8 | true | 标准模式；CP2/CP3/CP5 approve 前不执行复制。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | `.env`、data/reports、NAS、Git remote、runtime、data lake | true | `.env` 默认排除；reports/data/.venv/node_modules 排除；外部动作全部 not-authorized。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | process/docs/release/CR tracking | true | 新增 CR071 gate docs、manifest、copy plan 和后续执行验证证据。 |

## 回退决策

- 影响范围：本机文件系统新增 shadow root；旧目录保留，默认可通过删除新目录回退，但删除新目录不在本 CR 当前预授权内。
- 回退到阶段：`pre-CR071 local_backtest shadow-copy baseline`。
- 回退方式：
  - 若 CP2/CP3/CP5 前发现问题：不执行复制，CR071 保持 pending 或回退 candidate。
  - 若复制中断：保留旧目录，标记新目录为 partial-copy，先停止并进入人工决策；不得自动删除。
  - 若复制后发现排除项误入：停止后续切换，生成 CP7 finding；是否删除目标误入内容需用户单独授权。
- 需要重新确认的对象：
  - 目标目录若已存在且非空。
  - 是否复制 `.git`。
  - 是否允许把 `.env` 从 exclude 改为 include。
  - 是否扩展排除清单到 `runs/`、`.pytest_cache/` 或缓存目录。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及本机项目根迁移与回退策略。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 目标 root 从 `local_backtest` shadow 到 `quant-lab`。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 影响仓库根、Git 工作树和排除清单验证。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要明确 copy-first、shadow root、排除清单、回退和不授权边界。 |
| 是否保持 fast-lane | false | standard。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- 替代门禁：`CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS`
- 理由：本 CR 是单一受控本机复制操作，无代码 Story / 接口实现；CP5 以 execution readiness checkpoint 承载可实现性、排除清单、preflight 和回退策略。
- 开发 / 执行启动条件：
  - [ ] CP2-CR071 人工确认 `approved`
  - [ ] CP3-CR071 人工确认 `approved`
  - [ ] CP5-CR071 人工确认 `approved`
  - [ ] 执行前目标目录仍不存在或为空
  - [ ] `rsync` 可用
  - [ ] 排除清单仍为 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env`

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR071 与 CP2/CP3/CP5 门禁 | 用户请求、CR053-CR070 迁移治理基线 | 本 CR、contexts、checks、checkpoints、manifest | 不执行复制 | 等待用户 approve / 修改 / reject |
| 2 | `host-orchestrator` | 回填 CP2/CP3/CP5 approved | 用户回复 `approve` | 三份 checkpoint 人工审查结果、STATE / CR-INDEX 更新 | 仍不触碰 NAS / remote Git / runtime | 执行 preflight |
| 3 | `host-orchestrator` | 执行 copy-first shadow root | CR071 manifest、CP5 approved、preflight PASS | `/home/hyde/workspace/quant-lab` 新目录 | 只允许本机 `mkdir -p` + `rsync -a` 排除清单 | 写 CP6 执行证据 |
| 4 | `host-orchestrator` | 验证复制结果 | 目标目录、manifest | CP7 验证报告 / checks | 排除项不存在、`.git` 可读、旧目录保留 | 准备 CP8 |
| 5 | `host-orchestrator` | 收敛 CP8 | CP6/CP7 证据 | release readiness / feedback | 用户终验 | 关闭 CR071 或记录风险 |

## 执行命令草案

> 仅在 CP2/CP3/CP5 均 approved 且 preflight PASS 后允许执行。

```bash
mkdir -p /home/hyde/workspace/quant-lab
rsync -a \
  --exclude '/reports/' \
  --exclude '/data/' \
  --exclude '/.venv/' \
  --exclude '/node_modules/' \
  --exclude '/.env' \
  /home/hyde/workspace/local_backtest/ \
  /home/hyde/workspace/quant-lab/
```

## 自动终验授权

- 是否启用：false
- 授权范围：N/A
- 适用检查点：CP8
- 授权原文：N/A
- 回填要求：CP8 仍需后续人工确认；CP2/CP3/CP5 approve 只授权本机 copy-first 执行，不表示 CR071 自动关闭。

## 不授权项

| Item ID | 不授权操作 |
|---|---|
| NA-CR071-001 | 删除、重命名或移动 `/home/hyde/workspace/local_backtest` |
| NA-CR071-002 | 复制 `reports/`、`data/`、`.venv/`、`node_modules/` 或 `.env` |
| NA-CR071-003 | 读取、打印、记录或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实 |
| NA-CR071-004 | NAS mount / mkdir / copy / delete / archive promote / 迁移 |
| NA-CR071-005 | Git remote add/set-url/push/tag、branch/default branch governance、history rewrite、force push、remote deletion |
| NA-CR071-006 | `MARKET_DATA_LAKE_ROOT` 替换、data lake root 切换、provider fetch、lake write、catalog publish |
| NA-CR071-007 | QMT / MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live |
| NA-CR071-008 | 自动恢复或推进 CR046 |
| NA-CR071-009 | 对目标 `/home/hyde/workspace/quant-lab` 执行删除清理；若复制失败需另行授权 |

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：本 CR §后续事项台账；若 CP8 产生新项再拆独立 tracking
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| FU-CR071-01 | Post-copy root cutover / shell workflow switch | candidate | CR | 1 |  | source=CR071 | not-started | CR071 关闭后仍需独立决定是否把日常工作切到新 root | 用户明确要求后再启动 |
| FU-CR071-02 | Old root retention / retirement policy | candidate | CR | 1 |  | source=CR071 | not-started | 旧根按 MIG-A 保留，删除/归档属于破坏性或治理动作 | 用户明确要求后再启动 |
| FU-CR071-03 | Target env rebuild and smoke | candidate | CR | 2 |  | source=CR071 | not-started | target 不含 `.venv`、`.env`、data/reports，不能默认为可运行环境 | 需要 target 运行测试时再启动 |
| FU-CR071-04 | Data lake root / reports / run artifacts placement | candidate | CR | 2 |  | source=CR071 | not-started | CR071 不复制 data/reports，也不切换数据湖 | 另起 data lake / artifact migration gate |
| FU-CR071-05 | Remote Git sync / governance | candidate | CR | 2 |  | source=CR071 | not-started | CR071 不做远端 Git 写入或 branch/default branch 治理 | 另起 Git runtime authorization |

## 处理结论

- 审批结论：`closed-current-delivery / READY_WITH_RISK`
- [ ] 自动批准（低风险）
- [x] CP2 / CP3 / CP5 已人工确认
- [x] copy-first 已执行
- [x] CP6 / CP7 已通过，结论 `PASS_WITH_RISK`
- [x] CP8 delivery readiness 已经用户回复“好的关闭cr”确认

## 执行结果

| 项目 | 结果 |
|---|---|
| 执行时间 | 2026-06-16T07:33:48+08:00 |
| 执行模式 | local copy-first rsync |
| source root | `/home/hyde/workspace/local_backtest` |
| target root | `/home/hyde/workspace/quant-lab` |
| rsync files | 7,926 total / 7,415 regular files transferred |
| transferred size | 94,071,175 bytes |
| target size | 109M |
| old root retained | PASS |
| excluded paths absent | PASS：`reports/`、`data/`、`.venv/`、`node_modules/`、`.env` 均不存在 |
| `.git` copied | PASS |
| CP6 | `process/checks/CP6-CR071-COPY-FIRST-SHADOW-ROOT-CODING-DONE.md` / PASS_WITH_RISK |
| CP7 | `process/checks/CP7-CR071-COPY-FIRST-SHADOW-ROOT-VERIFICATION-DONE.md` / PASS_WITH_RISK |
| residual risk | target 仍不是完整 runtime/data/report mirror；root cutover、old root retirement、data/reports/env/runtime/remote Git 均后移且未授权 |

## CP8 关闭结果

| 项目 | 结果 |
|---|---|
| CP8 自动预检 | `process/checks/CP8-CR071-DELIVERY-READINESS.md` / PASS_WITH_RISK |
| CP8 人工检查点 | `process/checkpoints/CP8-CR071-DELIVERY-READINESS.md` / approved |
| Release Context | `process/release/RELEASE-CONTEXT-CR071.yaml` / READY_WITH_RISK |
| 用户确认 | 2026-06-16T09:13:26+08:00 回复“好的关闭cr” |
| reports Git 风险 | mitigated-before-cp8：source `45a426d`、target `2aaf21b` 均已停止 Git 跟踪 reports，文件保留 |
| target 排除项验证 | PASS：target 对 `reports data .venv node_modules .env` 的 `git status`、`git ls-files` 和根目录查找均为空 |
| remote Git | 只读核对 master=`3ade165e8b1edad031a911490cf6c1cee942616e`；未执行写动作 |
| CR072 归档 | reports/data/process evidence 已归档到 NAS 并校验；本地 reports/data 保留 |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | `CR-053` | quant-lab migration inventory / dry-run |
| CR | `CR-060` | repo-local identity convergence |
| CR | `CR-061` | package/import/layout convergence |
| CR | `CR-066`..`CR-070` | clean publication、ledger cleanup、Git governance 门禁 |
| CR | `CR-046` | 用户暂停；CR071 不恢复 |
| DOC | `docs/release/COPY-FIRST-SHADOW-ROOT-MIGRATION-PLAN-CR071.md` | CR071 执行计划 |
| MANIFEST | `docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml` | CR071 可执行清单 |
