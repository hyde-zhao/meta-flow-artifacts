---
cr_id: "CR-073"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "涉及候选工作根证据收敛、CR tracking、checkpoint/release/context 同步和 cutover readiness gate，必须保留 CP2/CP3/CP5 人工门禁。"
rollback_to: "pre-CR073 target evidence convergence baseline"
approval_result: "approved"
created_at: "2026-06-16T10:07:45+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-16T21:59:01+08:00"
closed_by: "user"
closed_at: "2026-06-17T15:46:35+08:00"
release_decision: "READY_WITH_RISK"
execution_result: "closed-current-delivery-ready-with-risk"
source: "user"
linked_issue: ""
parent_cr: "CR-071, CR-072"
source_checkpoint: "process/checkpoints/CP8-CR071-DELIVERY-READINESS.md"
source_decision_id: "USER-20260616-START-CR073-MIG-B"
follow_up_type: "target-root-convergence-cutover-readiness"
risk_class: "local_filesystem_evidence_sync"
owner: "host-orchestrator"
revisit_condition: "用户要求 root cutover、切换日常工作根、删除/重命名/移动旧根、复制 reports/data/.venv/node_modules/.env、读取凭据、远端 Git 写入、data lake 切换、QMT/MiniQMT runtime、CR046 recovery、使用 --delete 或目标冲突分类出现阻断时重访。"
acceptance_criteria: "CP2/CP3/CP5 approved 后，仅用 scoped local sync 将 CR071 CP8、CR072、CR073、STATE、CR-INDEX、release/context/checkpoint/check evidence 从旧根同步到 /home/hyde/workspace/quant-lab；排除 .git、reports/、data/、.venv/、node_modules/、.env；先 dry-run 与冲突分类；不使用 --delete；验证 target 最新 CR tracking / CP8 证据和排除项为空。"
close_condition: "target evidence convergence 完成并通过静态验证；仅达到 cutover readiness，不执行真正 root cutover。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-073 Target Root Convergence / Cutover Readiness Gate (MIG-B)

## 变更描述

用户要求启动 `CR073 / MIG-B`：在不切换日常工作根、不删除/重命名/移动旧根、不复制 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env`、不读取凭据、不做远端 Git 写入、不切 data lake、不触发 QMT/MiniQMT runtime、不恢复 CR046 的前提下，先对 `/home/hyde/workspace/quant-lab` 做 target evidence convergence。

本 CR 的目标不是运行 target，也不是发布 target，而是让 target 具备可审计的 CR tracking、CR071 CP8、CR072 NAS archive 和 CR073 门禁证据，为后续独立 root cutover CR 提供 readiness baseline。

## CR 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 与 CR046 影响面重叠 | false | CR046 是 QMT/MiniQMT runtime 线，保持用户暂停；CR073 不触发连接、凭据、账户、订单、simulation/live。 |
| 与 CR071 / CR072 关系 | continuation | CR073 消费 CR071 closed-current-delivery 与 CR072 NAS 归档结果，只做 target 证据收敛。 |
| 与远端 Git governance 重叠 | false | 不执行 `git push`、branch/default branch、tag、history rewrite、remote set-url/delete。 |
| 与 data lake / reports/data 恢复重叠 | false | 不复制或恢复 `reports/` / `data/`，不切 `MARKET_DATA_LAKE_ROOT`。 |
| 是否允许并行于 CR046 paused | true | 影响面为本机 evidence sync；CR046 继续 paused，不恢复。 |

## 初始目标差异扫描

| 类别 | 数量 | 代表路径 |
|---|---:|---|
| target 缺失 | 14 | `process/checkpoints/CP8-CR071-DELIVERY-READINESS.md`、`process/changes/CR-072-NAS-ARCHIVE-LOCAL-ASSETS-2026-06-16.md`、`docs/release/LOCAL-ASSETS-NAS-ARCHIVE-MANIFEST-CR072.yaml` |
| target 过期 | 6 | `process/STATE.md`、`process/changes/CR-INDEX.yaml`、`process/changes/CR-071-COPY-FIRST-SHADOW-ROOT-MIGRATION-2026-06-16.md` |
| target 一致 | 17 | CR071 CP2/CP3/CP5、CP6/CP7 与部分 docs/release / docs/quality 证据 |
| 排除项根目录查找 | 0 | target 根目录未发现 `reports data .venv node_modules .env` |
| 排除项 Git status | 0 | `git -C target status --short -- reports data .venv node_modules .env` 输出为空 |
| 排除项 Git ls-files | 0 | `git -C target ls-files -- reports data .venv node_modules .env` 输出为空 |

详细 dry-run 与分类见 `process/checks/CR073-TARGET-EVIDENCE-DIFF-2026-06-16.md`。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-073-TARGET-ROOT-CONVERGENCE-CUTOVER-READINESS-GATE-2026-06-16.md` | 新增 | N/A | 本文件 | approved |
| `process/context/CP2-CR073-REQUIREMENT-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/context/CP3-CR073-DESIGN-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/context/CP5-CR073-SCOPED-SYNC-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/checks/CR073-TARGET-EVIDENCE-DIFF-2026-06-16.md` | 新增 | N/A | 自动检查证据 | approved |
| `process/checks/CP2-CR073-TARGET-EVIDENCE-BASELINE.md` | 新增 | N/A | 自动预检 | approved |
| `process/checks/CP3-CR073-SCOPED-SYNC-HLD-CONSISTENCY.md` | 新增 | N/A | 自动预检 | approved |
| `process/checks/CP5-CR073-SCOPED-SYNC-READINESS.md` | 新增 | N/A | 自动预检 | approved |
| `process/checkpoints/CP2-CR073-TARGET-EVIDENCE-BASELINE.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checkpoints/CP3-CR073-SCOPED-SYNC-HLD-REVIEW.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checkpoints/CP5-CR073-SCOPED-SYNC-READINESS.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checks/CP2-CR073-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP2 人工门禁发起消息 | approved |
| `process/checks/CP3-CR073-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP3 人工门禁发起消息 | approved |
| `process/checks/CP5-CR073-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP5 人工门禁发起消息 | approved |
| `process/checks/CR073-SCOPED-SYNC-EXECUTION-2026-06-16.md` | 新增 | N/A | post-approval execution evidence | PASS |
| `process/checks/CR073-TARGET-CUTOVER-READINESS-VERIFICATION-2026-06-16.md` | 新增 | N/A | post-sync verification evidence | PASS |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused、CR071/CR072 closed 基线；追加 CR073 active gate | frontmatter / history | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR046 paused 与 closed CR 列表；追加 CR073 active gate | `active_crs` | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR071 CP8 closed-current-delivery | target CR tracking / CP8 evidence | 原文保留并同步副本到 target | target 当前缺少 CP8 closure evidence，CR073 只补齐证据，不重新打开 CR071。 |
| CR072 NAS archive closed-current-delivery | target CR072 formal CR / verification / manifest | 原文保留并同步副本到 target | target 当前缺少 CR072 证据，CR073 只同步 docs/process 证据，不做 NAS 新写入。 |
| `/home/hyde/workspace/local_backtest` | `/home/hyde/workspace/quant-lab` | 旧根继续保留 | CR073 不切日常工作根，仅确认 target 可作为后续 cutover 候选。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | MIG-B target evidence convergence | true | 新增 CR073 CP2 范围基线；不改产品需求正文。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | cutover readiness 静态验证 | true | 新增 target evidence diff、dry-run、排除项验证场景；不运行 target。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CP2/CP3/CP5 -> scoped sync -> validation | true | 标准模式；CP2/CP3/CP5 approve 前不得实际同步。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | `.env`、reports/data、旧根、远端 Git、data lake、runtime | true | 明确不授权；同步命令排除全部敏感/运行态目录且不使用 `--delete`。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | CR tracking、checkpoint、release/context evidence | true | 生成 CR073 门禁；通过后同步证据并验证 target。 |

## 回退决策

- 影响范围：局部，本机 target evidence 文件同步。
- 回退到阶段：`pre-CR073 target evidence convergence baseline`。
- 需要重新确认的对象：
  - CP2 scope：target evidence convergence 是否仍为本轮目标。
  - CP3 design：scoped local sync 是否仍为主方案。
  - CP5 readiness：dry-run、冲突分类、排除项验证是否仍通过。
- 回退行为：实际同步前可直接 reject；同步后若发现证据错误，不自动删除 target，不使用 `--delete`，另起修正 CR 或按用户授权恢复具体文件。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及候选 root 证据收敛和 cutover readiness。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 明确 root/cutover readiness 影响面与不授权边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不改代码接口，不进入 Story 实现。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要冻结 scoped sync 设计、失败路径和验证口径。 |
| 是否保持 fast-lane | false | 使用 standard CP2/CP3/CP5。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- 替代门禁：`CP5-CR073-SCOPED-SYNC-READINESS`
- 理由：CR073 是单一受控本机 evidence sync，不修改代码、接口、Schema 或 Story 实现；使用 CP5 execution readiness 替代 Story LLD 批次。
- 开发 / 同步启动条件：
  - [ ] `process/checkpoints/CP2-CR073-TARGET-EVIDENCE-BASELINE.md` 结论 `approved`
  - [ ] `process/checkpoints/CP3-CR073-SCOPED-SYNC-HLD-REVIEW.md` 结论 `approved`
  - [ ] `process/checkpoints/CP5-CR073-SCOPED-SYNC-READINESS.md` 结论 `approved`
  - [ ] 最新 `rsync -ain --relative` dry-run 无 forbidden path、无 `--delete`
  - [ ] target 排除项 `git status` / `git ls-files` / 根目录查找为空

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR073 与差异证据 | 用户请求、CR071 CP8、CR072 | 本 CR、diff check、contexts | 不写 target | 生成 CP2/CP3/CP5 |
| 2 | `host-orchestrator` | 发起 CP2/CP3/CP5 人工门禁 | 自动预检、Decision Brief | 三份 checkpoint、launch message | 等待用户 `approve` | 通过后执行 scoped sync |
| 3 | `host-orchestrator` | scoped local sync | CP2/CP3/CP5 approved、dry-run | target 证据文件更新 | 不使用 `--delete`，排除 forbidden paths | 写 CP6/CP7 或等价执行/验证证据 |
| 4 | `host-orchestrator` | 验证 target convergence | target 文件、Git 状态 | target validation evidence | 排除项为空，CR tracking 最新 | CR073 cutover readiness 收敛 |

## 同步方案

### 允许同步范围

- `process/STATE.md`
- `process/changes/CR-INDEX.yaml`
- `process/changes/CR-071-*`
- `process/changes/CR-072-*`
- `process/changes/CR-073-*`
- `process/checkpoints/*CR071*`、`*CR072*`、`*CR073*`
- `process/checks/*CR071*`、`*CR072*`、`*CR073*`
- `process/context/*CR071*`、`*CR072*`、`*CR073*`
- `process/release/*CR071*`、`*CR072*`、`*CR073*`
- `docs/release/*CR071*`、`*CR072*`、`*CR073*`
- `docs/quality/*CR071*`、`*CR072*`、`*CR073*`

### 冻结命令形态

```bash
rsync -ain --relative \
  --exclude='.git/' \
  --exclude='reports/' \
  --exclude='data/' \
  --exclude='.venv/' \
  --exclude='node_modules/' \
  --exclude='.env' \
  --files-from=<(find process docs -type f \( \
    -path 'process/STATE.md' \
    -o -path 'process/changes/CR-INDEX.yaml' \
    -o -name '*CR071*' -o -name '*CR-071*' \
    -o -name '*CR072*' -o -name '*CR-072*' \
    -o -name '*CR073*' -o -name '*CR-073*' \
  \) | sort) \
  ./ /home/hyde/workspace/quant-lab
```

实际同步仅在 `-n` 去掉后执行，仍保留相同 include/exclude，不添加 `--delete`。

## 不授权项

| ID | 不授权内容 |
|---|---|
| NA-CR073-001 | 切换日常 shell / IDE / automation / systemd / cron 工作根到 `/home/hyde/workspace/quant-lab` |
| NA-CR073-002 | 删除、重命名或移动 `/home/hyde/workspace/local_backtest` |
| NA-CR073-003 | 复制、恢复、读取或比对 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env` |
| NA-CR073-004 | 读取、打印、记录或迁移 token、密码、cookie、session、private key、账户或交易事实 |
| NA-CR073-005 | 使用 `--delete`、自动清理 target、覆盖未知冲突或执行 destructive cleanup |
| NA-CR073-006 | 远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url |
| NA-CR073-007 | NAS 新写入 / 删除 / promote，data lake root switch、provider fetch、lake write、catalog publish |
| NA-CR073-008 | QMT / MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live |
| NA-CR073-009 | 自动恢复或推进 CR046 |

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：本 CR §后续事项台账；CR073 收敛时可拆独立 follow-up tracking
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`python3 -B scripts/check_cr_tracking_consistency.py --project-root .`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR074-candidate | Root cutover / daily workflow switch | candidate | CR | 1 |  | source=CR073 | not-started | CR073 只做到 cutover readiness | 用户明确要求后再启动 |
| CR075-candidate | Target env rebuild and smoke | active | CR | 2 | `process/changes/CR-075-TARGET-ENV-REBUILD-AND-SMOKE-2026-06-16.md` | source=CR073/CR074 | CP2/CP3/CP5 human gate | env rebuild / smoke 仍需人工门禁 approve | 已启动正式 CR075，等待 CP2/CP3/CP5 审查 |
| CR076-candidate | Data/reports access policy | candidate | CR | 3 |  | source=CR073 | not-started | reports/data 不复制 | 后续决定 NAS / local retained / rebuild 策略 |
| CR077-candidate | Old root retirement policy | candidate | CR | 4 |  | source=CR073 | not-started | 旧根删除/移动是破坏性动作 | target authoritative 后再评审 |
| CR078-candidate | Remote Git governance after cutover | candidate | CR | 5 |  | source=CR073 | not-started | 远端写入不在 CR073 范围 | 后续独立 gate |

## 处理结论

- 审批结论：approved。用户于 2026-06-16T21:59:01+08:00 回复“同意”，按 CP2/CP3/CP5 approve 处理。
- 执行结论：PASS。scoped local sync 已执行；post-sync dry-run 无输出，逐文件 `cmp` 无差异，target forbidden paths 三项检查为空。
- CR086 收敛结论：closed-current-delivery / READY_WITH_RISK。用户于 2026-06-17T15:46:35+08:00 回复“同意”，批准 CR086 ledger-only 收敛；CR073 target evidence convergence / cutover readiness 已达到当前交付目标，后续 root cutover、data、runtime、remote governance、old root retirement 和 CR046 recovery 均保持独立门禁。
- [ ] 自动批准（低风险）
- [x] 待人工确认（中风险）
- [ ] 待人工审批（高风险）

## CR086 收敛结果

CR086 仅对过程台账做状态收敛，不新增本 CR 的文件同步、root 切换、数据、运行时或远端动作。CR073 当前收敛为 `closed-current-delivery / READY_WITH_RISK`：已完成 target evidence convergence / cutover readiness 目标，但不表示授权真正 root cutover、旧根删除 / 移动、forbidden paths 复制 / 读取、凭据、远端 Git、NAS 新写入、data lake、QMT / MiniQMT runtime 或 CR046 recovery。

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | `CR-071` | Copy-first shadow root 已 CP8 关闭；target 当前缺少 CP8 closure evidence。 |
| CR | `CR-072` | NAS archive 已关闭；target 当前缺少 CR072 evidence。 |
| Context | `process/context/NEXT-SESSION-CR073-STARTUP-CONTEXT.md` | CR073 启动上下文。 |
| Target root | `/home/hyde/workspace/quant-lab` | 本轮只收敛证据，不切工作根。 |
