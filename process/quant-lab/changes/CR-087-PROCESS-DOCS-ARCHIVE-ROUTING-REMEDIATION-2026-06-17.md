---
cr_id: "CR-087"
status: "closed-local-remediation"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中文档归档路由、软链接与文件所有权边界调整，按 standard 记录。"
rollback_to: "CR082 process ledger namespace baseline"
approval_result: "completed"
created_at: "2026-06-17T19:36:45+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-17T19:36:45+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-082"
source_checkpoint: "process/checkpoints/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md"
source_decision_id: "DQ-CP3-CR081-04"
follow_up_type: "process-docs-externalization"
risk_class: "medium"
owner: "host-orchestrator"
revisit_condition: "process docs or legacy checkpoint paths appear again as real in-repo directories"
acceptance_criteria: "process symlink is healthy; process metadata exists; docs/checkpoints process artifacts are archived under /home/hyde/workspace/process/quant-lab and old paths remain readable through symlinks"
close_condition: "local route checks pass and no data/reports/credential/runtime/remote action was performed"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR087 Process Docs Archive Routing Remediation

## 变更描述

用户指出 meta-flow 的过程文件归档方式已经变化，要求检查并整改本项目的过程文档与过程文档软链接位置。

本 CR 是 CR081 / CR082 后续的本地路由整改：CR081 已明确将 `docs/design`、`docs/features`、`docs/quality`、`docs/release` 外置归档 deferred；当前整改将这些过程 / 审计文档和旧根目录 `checkpoints/` 纳入外部 process namespace，并保留旧路径可读。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/STATE.md` | 原文档更新 | 既有状态保留，追加 artifact routing 字段 | `root_authority` / `artifact_routing` | approved |
| `process/.meta-flow-process.yaml` | 新增 | N/A | N/A | approved |
| `LEDGER.md` | 原文档更新 | 保留 CR081 / CR082 说明并追加 CR087 路由 | N/A | approved |
| `ledger.yaml` | 原文档更新 | 保留 process ledger route 并追加 docs/checkpoints route | N/A | approved |
| `scripts/link-engineering-ledger.sh` | 原文档更新 | 保留 process symlink bootstrap，追加 docs/checkpoints symlink bootstrap | N/A | approved |
| `.gitignore` | 原文档更新 | 保留既有忽略策略，追加过程文档软链和内部 release 归档忽略 | N/A | approved |
| `docs/design/` | 归档 | 内容归档到 `process/docs/design/`，旧路径改软链 | N/A | approved |
| `docs/features/` | 归档 | 内容归档到 `process/docs/features/`，旧路径改软链 | N/A | approved |
| `docs/quality/` | 归档 | 内容归档到 `process/docs/quality/`，旧路径改软链 | N/A | approved |
| `docs/release/*-CR*` / internal manifests | 归档 | 内容归档到 `process/docs/release/`，公开入口文件留在源码树 | N/A | approved |
| `checkpoints/` | 归档 | 内容归并到 `process/checkpoints/`，旧路径改软链 | N/A | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` | false | 不修改业务需求 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不修改验证场景 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | `process/DEVELOPMENT-PLAN.yaml` | false | 仅迁移过程文档路径 |
| 安全层 | 是否引入新的高风险动作或权限要求 | 文件系统路由 / symlink | true | 仅本地文件归档与软链；不触碰数据、报告、凭据、远端 Git、NAS 或 runtime |
| 交付层 | 是否需要重新生成交付物或回归子集 | `.gitignore`、ledger 指针、bootstrap、路由元数据 | true | 更新本地路由元数据并执行 symlink / readability 检查 |

## 回退决策

- 影响范围：局部，限过程文档归档路由与软链接。
- 回退到阶段：`CR082 process ledger namespace baseline`。
- 回退方式：移除新增 docs/checkpoints symlink，恢复迁移前真实目录备份；`process -> ../process/quant-lab` 保持不变。
- 需要重新确认的对象：仅当要删除外部归档或清理备份时需要单独人工授权。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及路径所有权和软链接 |
| 修改架构、权限、安全边界或平台安装路径 | true | 修改过程文档归档路由 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 修改过程文档文件所有权 |
| 需要 HLD / LLD 才能解释影响 | false | 由 CR081/CR082 已批准设计延伸 |
| 是否保持 fast-lane | false | standard 记录 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- 理由：不修改业务代码、接口契约、Story 实现或测试逻辑。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 路由健康检查 | 当前仓库、`process/STATE.md` | 本 CR | 用户请求 | 执行归档 |
| 2 | `host-orchestrator` | 迁移过程文档并重建软链 | docs/checkpoints、external process namespace | external docs archive、旧路径 symlink | 不触碰 forbidden paths | 更新元数据 |
| 3 | `host-orchestrator` | 更新路由说明与 bootstrap | `.gitignore`、`ledger.yaml`、`LEDGER.md`、script | 路由元数据 | 本地检查 | 验证 |
| 4 | `host-orchestrator` | 执行本地验证 | symlink、readability、Git status | `process/checks/CR087-*` | PASS / PASS_WITH_RISK | 关闭本地整改 |

## 自动终验授权

- 是否启用：true
- 授权范围：仅本地过程文档路由整改
- 适用检查点：本地 route check
- 自动通过条件：
  - [ ] `process/STATE.md` 可读
  - [ ] `process/.meta-flow-process.yaml` 存在
  - [ ] `docs/design`、`docs/features`、`docs/quality`、`checkpoints` 为可读软链
  - [ ] 内部 release 归档可通过旧路径读取
  - [ ] 未执行数据、报告、凭据、远端 Git、NAS、runtime 或 cleanup 动作
- 授权原文：用户要求“检查一下该项目的过程文档和过程文档的软连接位置，对其进行整改”
- 授权时间：2026-06-17T19:36:45+08:00

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：不新增独立台账；本 CR 记录剩余风险
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：本轮记录跳过原因或执行本地脚本

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR087-FU-001 | Git index 中历史已跟踪过程文档去跟踪治理 | candidate | CR | 1 |  | CR078 / CR085 publication governance | 未启动 | 需要单独 Git index / publication gate | 用户确认后单独处理 |

## 路由验证结果

| 检查项 | 结果 | 证据 |
|---|---|---|
| `process` 入口 | PASS | `/home/hyde/workspace/quant-lab/process -> ../process/quant-lab` |
| route metadata | PASS | `process/.meta-flow-process.yaml` 可读 |
| 外部 docs archive | PASS | `/home/hyde/workspace/process/quant-lab/docs`，252 个文件 |
| legacy checkpoints 合并 | PASS | `process/checkpoints` 当前 195 个文件，包含旧根目录 checkpoint |
| 旧 docs 路径可读 | PASS | `docs/design/HLD.md`、`docs/quality/VERIFICATION-REPORT-CR080.md`、`docs/release/RELEASE-NOTES-CR082.md` 可读 |
| 公开 release 入口 | PASS | `docs/release/RELEASE-NOTES.md` 等五个公开入口保留为真实文件 |
| 断链检查 | PASS | `find docs/design docs/features docs/quality docs/release checkpoints -xtype l` 无输出 |
| bootstrap 幂等性 | PASS | `scripts/link-engineering-ledger.sh` 返回 PASS |
| 禁止动作 | PASS | 未执行 data/reports 内容读取、凭据读取、NAS 内容操作、远端 Git 写、runtime 或 cleanup |

## 处理结论

- 审批结论：`completed`
- [x] 自动批准（本地过程文档路由整改）
- [x] 本地 route check 通过
- [x] 当前 CR 关闭为 `closed-local-remediation`
- [ ] 待人工确认（中风险）
- [ ] 待人工审批（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | CR-081 | process ledger externalization；docs externalization deferred |
| CR | CR-082 | process ledger project namespace normalization |
| 文件 | `process/.meta-flow-process.yaml` | 新增 route metadata |
| 文件 | `docs/design` / `docs/features` / `docs/quality` / `docs/release` / `checkpoints` | 过程文档归档与旧路径可读入口 |
