---
cr_id: "CR-080"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "触碰真实 reports/data restore/copy、目标根写入、内容读取边界和回滚边界，必须使用 standard。"
rollback_to: "CR079 closed-current-delivery / CR079-restore-candidate"
approval_result: "approved"
created_at: "2026-06-17T06:42:48+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-17T06:57:22+08:00"
preflight_result: "PASS"
preflight_at: "2026-06-17T07:03:43+08:00"
preflight_evidence: "process/checks/CR080-DATA-REPORTS-RESTORE-PREFLIGHT-2026-06-17.md"
execution_result: "PASS"
execution_started_at: "2026-06-17T07:04:55+08:00"
execution_completed_at: "2026-06-17T07:05:05+08:00"
execution_evidence: "process/checks/CR080-DATA-REPORTS-RESTORE-EXECUTION-2026-06-17.md"
cp6_result: "PASS"
cp6_evidence: "process/checks/CP6-CR080-DATA-REPORTS-RESTORE-CODING-DONE.md"
cp7_result: "PASS_WITH_RISK"
cp7_evidence: "process/checks/CP7-CR080-DATA-REPORTS-RESTORE-VERIFICATION-DONE.md"
cp8_result: "approved"
cp8_checkpoint: "process/checkpoints/CP8-CR080-DELIVERY-READINESS.md"
cp8_approved_by: "user"
cp8_approved_at: "2026-06-17T07:32:59+08:00"
delivery_status: "closed-current-delivery"
release_decision: "READY_WITH_RISK"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-079"
source_checkpoint: "process/checkpoints/CP8-CR079-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR079-04"
follow_up_type: "CR"
risk_class: "high"
owner: "host-orchestrator"
revisit_condition: "已满足并关闭：用户明确启动 CR079-restore-candidate，且 CR079 metadata-only inventory 证明 target root 缺少 reports/data、legacy root retained assets 存在。"
acceptance_criteria: "CP2/CP3/CP5 人工门禁 approved 后，允许执行受控本地 restore/copy preflight；若 preflight PASS，则仅按批准命令族从 legacy retained assets 向 target authoritative root 复制 reports/data，并用元数据摘要验证。"
close_condition: "restore/copy 执行与验证通过，或因冲突 / 风险 / 用户选择关闭为 blocked / cancelled / superseded；任何内容读取、凭据、NAS、provider/lake/catalog 或删除必须另起门禁。"
cr_index_path: "process/changes/CR-INDEX.yaml"
target_root: "/home/hyde/workspace/quant-lab"
legacy_root: "/home/hyde/workspace/local_backtest"
---

# CR-080 Data/reports Restore or Copy Gate

## 变更描述

用户明确要求启动 `CR079-restore-candidate`。本 CR 将该候选转为正式 CR080，用于审批真实 `reports/` 与 `data/` 从 legacy retained assets 到 authoritative target root 的受控 restore/copy。

本文件创建阶段不执行复制、恢复、删除、内容读取、内容 hash、NAS 访问、provider fetch、lake write、catalog publish、远端 Git 写入、QMT/MiniQMT runtime 或旧根退役。用户已于 `2026-06-17T06:57:22+08:00` 批准 CP2 / CP3 / CP5；post-approval preflight 已 PASS，并已在 `2026-06-17T07:04:55+08:00..07:05:05+08:00` 按批准命令族完成本地受控 copy。用户于 `2026-06-17T07:32:59+08:00` 回复“同意”，按 CR080 CP8 approve 处理，接受 `READY_WITH_RISK` 并关闭当前交付。该授权已消费，不表示继续授权重复 copy、删除、内容读取、NAS/provider/lake/catalog、远端 Git、runtime、CR046 recovery 或旧根退役。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-079` |
| 来源检查点 | `process/checkpoints/CP8-CR079-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR079-04` |
| follow-up 类型 | `CR` |
| 风险等级 | `high` |
| owner | `host-orchestrator` |
| 重访条件 | CR079 已关闭 metadata-only inventory，target root 缺少 `reports/` / `data/`，legacy root retained assets 存在。 |
| 验收标准 | 三门人工批准、preflight PASS、受控 copy/restore 执行、元数据验证 PASS、禁止项计数为 0。 |
| 关闭条件 | restore/copy 验证完成或明确 blocked / cancelled / superseded。 |

## 冲突预检

| 对象 | 影响面 | 结论 | 说明 |
|---|---|---|---|
| `CR-046` | QMT / MiniQMT strategy framework | no-overlap | CR080 不恢复 CR046、不启动 runtime、不读取凭据、不查询账户或交易事实。 |
| `CR-073` | target root convergence | continuation | CR080 承接 target root 已成为 authoritative root 的事实，只补齐未迁移的 `reports/` / `data/`。 |
| `CR-074` | logical root cutover | continuation | CR080 尊重 `/home/hyde/workspace/quant-lab` 为 target authoritative root；不切换当前 shell cwd。 |
| `CR-076` | data/reports policy | continuation | CR076 只建立 no-data-touch policy；CR080 是后续独立执行门禁。 |
| `CR-079` | metadata-only inventory | source | CR079 证明 target 缺失、legacy retained assets 存在；CR080 不重新打开 CR079。 |
| `CR077-candidate` | old root retirement | related-no-overlap | CR080 不删除、移动或重命名旧根；restore 完成后可为 CR077 提供输入。 |
| `CR078-candidate` | remote Git governance | no-overlap | CR080 不做远端 Git 写入、branch/default branch governance 或 publish。 |
| `CR079-nas-compare-candidate` | NAS compare | not-started | CR080 推荐先使用 legacy retained assets；NAS compare 仅在本地来源不足或用户要求一致性证明时另起。 |
| `CR079-provider-rebuild-candidate` | provider/lake/catalog rebuild | not-started | CR080 不执行 provider fetch、lake write 或 catalog publish。 |

冲突预检结论：允许创建正式 CR080 并发起 CP2/CP3/CP5 人工门禁；执行 restore/copy 前仍必须通过三门批准和 post-approval preflight。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | 新增 / 状态更新 | N/A | N/A | CP8 approved / closed-current-delivery |
| `process/STATE.md` | 原文档更新 | 保留 CR079 closure 和历史 history | `history` / `cr_tracking` / `checkpoints` | CP8 approval synced |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 `CR079-restore-candidate` 行并关闭来源候选 | `active_crs` / `follow_up_candidates` | CP8 approval synced |
| `process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml` | 新增 | N/A | N/A | pending |
| `process/context/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONTEXT.yaml` | 新增 | N/A | N/A | pending |
| `process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml` | 新增 | N/A | N/A | pending |
| `docs/product/*` | 不变 | 既有需求基线保留 | N/A | pending |
| `docs/design/*` | 不变 | 既有架构基线保留 | N/A | pending |
| `docs/release/*-CR079.md` | 不变 | CR079 已关闭，不重写 | N/A | pending |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| `CR079-restore-candidate` | `CR-080` | 台账保留候选来源，正式需求进入 CR080 | CR080 是 CR079 CP8 后续项的正式执行门禁。 |
| `CR079 metadata-only inventory` | CR080 preflight / execution evidence | 原文保留 | CR079 只证明 target 缺失和 legacy retained assets 存在，不证明 restore 完成。 |
| CR076 no-data-touch policy | CR080 copy/restore authorization | 原文保留 | CR080 只在三门批准后扩大到受控 restore/copy；凭据、内容读取、provider/lake/catalog 等仍禁止。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增或重定义 data/reports restore 需求 | `CR079-restore-candidate` / target root 可用性 | true | 创建 CR080，明确来源、目标、命令族、验证和回滚边界。 |
| 场景层 | 是否改变测试 / 验证范围 | target/legacy `reports/`、`data/` metadata verification | true | 增加 restore/copy preflight、post-copy metadata verification 和禁止项验证。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CP2/CP3/CP5 -> post-approval preflight -> execution -> CP7/CP8 | true | 三门批准前只允许门禁文件；批准后先 preflight，再受控执行。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | 本地真实资产、目标根写入、credential-like path 过滤 | true | 使用 standard；禁止内容读取、凭据、NAS/provider/lake/catalog、远端 Git、runtime 和删除。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | CR080 checks / verification / release context | true | 后续执行时生成 CP6 execution、CP7 verification、CP8 readiness 和 release docs。 |

## 回退决策

- 影响范围：局部，但触碰真实本地资产和 target root 写入，安全等级 high。
- 回退到阶段：`CR079 closed-current-delivery / CR079-restore-candidate`。
- 需要重新确认的对象：
  - CP2 范围基线：是否启动 CR080、来源/目标是否只使用 legacy retained assets 和 target root。
  - CP3 设计边界：命令族、冲突处理、exclude / denylist、证据粒度、失败路径。
  - CP5 执行授权：是否允许 post-approval preflight 和受控 restore/copy。
- 回退方式：三门未批准时只回滚 process ledger；执行阶段若 preflight 失败则不复制并关闭为 blocked；若 copy 中途失败，不自动删除或 cleanup，必须记录 partial 状态并另行确认清理策略。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 目标是恢复/复制真实本地资产。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 改变 data/reports 访问授权边界和 target root 写入状态。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不改代码接口或 Story。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要 CP3/CP5 说明命令族、冲突处理、验证和回滚。 |
| 是否保持 fast-lane | false | standard。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- 替代门禁：`CP5-CR080-DATA-REPORTS-RESTORE-READINESS`
- 理由：CR080 是 runtime authorization / execution gate，不修改代码、接口、Schema、Story 或文件 owner；使用 CP5 readiness 替代 Story LLD 批次。
- 批次人工确认稿：`process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md`
- 执行启动条件：
  - [ ] `process/checkpoints/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` 结论 `approved`
  - [ ] `process/checkpoints/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-REVIEW.md` 结论 `approved`
  - [ ] `process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` 结论 `approved`
  - [ ] CR tracking consistency PASS
  - [ ] Human-gate 校验 PASS
  - [ ] target `reports/` / `data/` 仍缺失或为空；若非空，必须转人工冲突处理
  - [ ] denylist / exclude policy 生效；credential-like path 不复制

## 推荐执行设计

推荐采用 legacy retained assets -> target root 的受控本地 copy-first 方案：

1. 三门批准前只创建门禁、上下文和审查材料。
2. 批准后先执行 post-approval preflight：检查 target `reports/` / `data/` 是否仍不存在或为空、legacy 来源是否存在、命令族是否在白名单内、denylist 是否生效、CR tracking 是否 PASS。
3. 若 preflight PASS，使用本地 copy/sync 命令族复制：
   - 来源：`/home/hyde/workspace/local_backtest/reports`、`/home/hyde/workspace/local_backtest/data`
   - 目标：`/home/hyde/workspace/quant-lab/reports`、`/home/hyde/workspace/quant-lab/data`
   - 允许命令族：`test`、`mkdir`、`find` / `stat` / `du` 元数据检查、`rsync` 本地复制（不得 `--delete`、不得 `--checksum`、不得跨远端）、必要的 `chmod` 不在推荐路径内。
   - 必须排除 `.env*`、`*token*`、`*secret*`、`*password*`、`*private*`、`*credential*`、`*.key` 等 credential-like path；证据中只写脱敏计数。
4. 验证只记录聚合元数据：存在性、文件/目录数、总字节数、mtime 范围、exclude 命中计数、禁止操作计数；不读取内容、不打印样本、不解析文件、不计算内容 hash。
5. 若 target 已有非空目录、出现命令越界、出现 credential-like 冲突或 preflight 不一致，立即 BLOCKED，不执行 copy。

## 不授权项

| ID | 不授权内容 |
|---|---|
| NA-CR080-001 | CP2/CP3/CP5 approved 前，不执行任何 `reports/` / `data/` 复制、恢复、删除、内容读取、内容 hash 或内容比对。 |
| NA-CR080-002 | 不读取文件内容、不打印样本、不解析 parquet/csv/json、不计算内容 hash。 |
| NA-CR080-003 | 不读取、打印、记录、复制或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。 |
| NA-CR080-004 | 不复制 credential-like path；命中项只能脱敏计数并进入风险记录。 |
| NA-CR080-005 | 不执行 NAS compare / promote、provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement 或 provider rebuild。 |
| NA-CR080-006 | 不执行远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。 |
| NA-CR080-007 | 不启动 QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live。 |
| NA-CR080-008 | 不删除、重命名或移动旧根 `/home/hyde/workspace/local_backtest`。 |
| NA-CR080-009 | 不恢复或推进 CR046。 |
| NA-CR080-010 | 不执行 optional dependency groups、full tests、删除 `.venv`、`--delete` 同步或 destructive cleanup。 |

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR080 并发起三门审查 | 用户请求 / CR079 CP8 follow-up | 本 CR、context、CP2/CP3/CP5 checks/checkpoints | CR tracking PASS | 等待用户确认 |
| 2 | `host-orchestrator` | post-approval preflight | 三门 approved、CR080 | preflight check | target 冲突和命令边界 PASS | 执行或 BLOCKED |
| 3 | `host-orchestrator` | 受控 restore/copy | preflight PASS | copy execution evidence | 禁止项计数为 0 | 进入验证 |
| 4 | `host-orchestrator` | 元数据验证 | copy evidence | CP7 verification | PASS / PASS_WITH_RISK / BLOCKED | 准备 CP8 |
| 5 | `host-orchestrator` | 收敛终验 | CR080 证据 | CP8 自动预检和人工审查稿 | 用户确认 | 关闭 CR080 或回退 |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：N/A
- 自动通过条件：N/A
- 授权原文：N/A

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR079-restore-candidate | Data/reports restore or copy gate | closed-current-delivery | CR | 1 | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | closed_by=CR-080 | closed | 用户已 approve CR080 CP8 | 已完成 preflight、受控本地 copy、CP6、CP7 和 CP8；当前交付关闭。 |
| CR079-nas-compare-candidate | NAS archive compare gate | candidate | CR | 2 |  | blocked_by=CR080 outcome | not-started | 本轮不访问 NAS；只有 legacy retained assets 不足或需要归档一致性证明时启动 | 暂不启动。 |
| CR079-provider-rebuild-candidate | Provider rebuild gate | candidate | CR | 3 |  | blocked_by=CR080 / NAS outcome | not-started | 本轮不执行 provider fetch/lake/catalog | 暂不启动。 |

## 处理结论

- 审批结论：`approved`（CP2/CP3/CP5：用户回复“同意”，2026-06-17T06:57:22+08:00；CP8：用户回复“同意”，2026-06-17T07:32:59+08:00）
- 当前门禁：CP2/CP3/CP5 approved；post-approval preflight PASS；受控本地 copy PASS；CP6 PASS；CP7 PASS_WITH_RISK；CP8 approved；当前交付关闭为 `closed-current-delivery / READY_WITH_RISK`。
- [ ] 自动批准（低风险）
- [ ] 待人工确认（中风险）
- [x] CP2/CP3/CP5 人工审批已通过（高风险）
- [x] CP8 人工终验已通过（READY_WITH_RISK）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | `CR-079` | metadata-only inventory 来源。 |
| Candidate | `CR079-restore-candidate` | 本 CR 来源候选。 |
| Target root | `/home/hyde/workspace/quant-lab` | authoritative root；CR080 已补齐 `reports/` / `data/`，等待 CP6/CP7/CP8 收敛。 |
| Legacy root | `/home/hyde/workspace/local_backtest` | retained assets 来源；CR080 不删除、不移动、不重命名。 |
