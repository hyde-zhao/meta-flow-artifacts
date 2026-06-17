---
cr_id: "CR-072"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "NAS 写入与本地缓存清理属于运行授权 / 本地资产治理动作，按 standard 留痕。"
rollback_to: "pre-cr072-local-assets-state"
approval_result: "approved-by-user-current-turn"
created_at: "2026-06-16T08:19:50+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-16T08:19:50+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-071"
source_checkpoint: "process/checks/CP7-CR071-COPY-FIRST-SHADOW-ROOT-VERIFICATION-DONE.md"
source_decision_id: "USER-20260616-ARCHIVE-LOCAL-ASSETS"
follow_up_type: "local-asset-archive"
risk_class: "medium"
owner: "host-orchestrator"
revisit_condition: "NAS 归档失败、manifest 校验失败、用户要求删除本地 reports/data、或需要将过程证据纳入远端发布。"
acceptance_criteria: "reports/data/过程证据已复制到 NAS；manifest 和 checksum 已生成；本地 reports/data 保留；可重建缓存已清理；无远端 Git 写入。"
close_condition: "closed at 2026-06-16T08:19:50+08:00 execution window: NAS archive PASS, checksum PASS, cache cleanup PASS_WITH_RISK."
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-072 NAS Archive Local Assets

## 变更描述

用户要求“按照你的建议处理”当前仓库中未提交 / 未推送到远端的文件。按前置检查结论，本 CR 执行以下动作：

- 将本地 `reports/` 归档到 NAS。
- 将本地 `data/` 小型数据快照归档到 NAS。
- 将 CR058-CR071 相关 `docs/`、`process/`、`ops/` 过程证据归档到 NAS。
- 生成 NAS 侧 manifest、size、checksum 和执行摘要。
- 清理可重建缓存：`.venv/`、`node_modules/`、`.pytest_cache/`、`__pycache__/`。

## 明确不授权项

- 不删除本地 `reports/`。
- 不删除本地 `data/`。
- 不执行远端 Git 写入、push、tag、branch/default branch 变更、history rewrite 或 force push。
- 不执行 data lake root 切换、catalog publish、provider fetch、QMT / MiniQMT runtime、凭据读取或 CR046 recovery。
- 不把本轮归档动作解释为 CR071 CP8 自动关闭。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-072-NAS-ARCHIVE-LOCAL-ASSETS-2026-06-16.md` | 新增 | N/A | N/A | approved |
| `docs/release/LOCAL-ASSETS-NAS-ARCHIVE-MANIFEST-CR072.yaml` | 新增 | N/A | N/A | approved |
| `docs/quality/VERIFICATION-REPORT-CR072.md` | 新增 | N/A | N/A | approved |
| `process/checks/CP6-CR072-NAS-ARCHIVE-EXECUTION-DONE.md` | 新增 | N/A | N/A | approved |
| `process/checks/CP7-CR072-NAS-ARCHIVE-VERIFICATION-DONE.md` | 新增 | N/A | N/A | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| 本地 `reports/` | NAS reports archive | 原文保留 + NAS 副本 | 本地副本不删除；NAS 副本用于长期保留和后续可选清理依据。 |
| 本地 `data/` | NAS data snapshot archive | 原文保留 + NAS 副本 | 本地副本不删除；NAS 副本用于复现实验和缓存恢复。 |
| 本地 `docs/` / `process/` / `ops/` CR058-CR071 证据 | NAS evidence archive | 原文保留 + NAS 副本 | 本地工作树仍保留，NAS 副本作为审计证据包。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | N/A | false | 不改需求基线。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | N/A | false | 不改测试矩阵。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR071 后续处理 | true | 作为 CR071 后的本地资产治理动作，独立留痕，不关闭 CR071。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | NAS 写入、本地缓存删除 | true | 仅执行用户本轮授权范围；不删除不可重建资产，不做远端 Git / runtime / 凭据动作。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | 归档 manifest / 验证报告 | true | 生成 CR072 manifest、CP6/CP7 和 verification report。 |

## 回退决策

- 影响范围：局部。
- 回退到阶段：`pre-cr072-local-assets-state`。
- 需要重新确认的对象：
  - 若 NAS 归档有误，可删除 NAS 侧 CR072 归档目录后重跑归档。
  - 若缓存清理影响本地运行，使用 `uv sync` 重建 `.venv/`；`node_modules/` 不作为项目入口，默认不重建。
  - 本地 `reports/` 与 `data/` 未删除，无需恢复。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及 NAS 写入和缓存删除。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 涉及 NAS 写入授权边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不改代码契约。 |
| 需要 HLD / LLD 才能解释影响 | false | 使用现有 CR059 NAS directory map。 |
| 是否保持 fast-lane | false | 保持 standard 留痕。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- 理由：本 CR 不修改代码、接口契约、Story 实现或 dev gate。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR072 | 用户授权 / 前置检查 | CR072 | 用户本轮授权 | 执行归档 |
| 2 | `host-orchestrator` | 复制 reports/data/证据到 NAS | 本地目录 / CR059 NAS map | NAS 归档目录 | rsync 退出码 0 | 生成 manifest |
| 3 | `host-orchestrator` | 生成 manifest/checksum | NAS 归档目录 | manifest / checksum | 文件数与大小可核验 | 清理缓存 |
| 4 | `host-orchestrator` | 清理可重建缓存 | `.venv/`、`node_modules/`、`.pytest_cache/`、`__pycache__/` | 本地缓存释放 | 不触碰 reports/data | 验证 |
| 5 | `host-orchestrator` | 验证并收敛 | manifest / git status / du | CP6/CP7 / verification report | PASS 或 PASS_WITH_RISK | 向用户汇报 |

## 自动终验授权

- 是否启用：true
- 授权范围：仅本 CR 的归档执行和可重建缓存清理。
- 适用检查点：CP6 / CP7。
- 自动通过条件：
  - [x] NAS 归档命令成功。
  - [x] manifest 和 checksum 文件生成。
  - [x] 本地 `reports/` / `data/` 仍存在。
  - [x] 未执行远端 Git 写入。
- 授权原文：用户回复“按照你的建议处理”。
- 授权时间：2026-06-16T08:19:50+08:00。
- 回填要求：验证文件标注 `approval_source=user-current-turn`。

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：N/A，本 CR 内记录。
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：已同步 `process/STATE.md.cr_tracking.closed_crs[]` 与 `process/changes/CR-INDEX.yaml.closed_crs[]`。

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| FU-CR072-01 | 本地 `reports/` / `data/` 删除或瘦身 | candidate | CR | 2 | N/A | CR072 | 未启动 | 需要用户单独授权和恢复验证 | 如用户需要释放本地空间，再发起删除门禁。 |
| FU-CR072-02 | 旧根 `local_backtest` 与新根 `quant-lab` 收敛 | candidate | CR | 1 | N/A | CR071 | 未启动 | CR071 CP8 未关闭 | CR071 关闭后决定工作根切换 / 旧根保留策略。 |

## 执行结果

| 对象 | 结果 | 证据 |
|---|---|---|
| `reports/` NAS 归档 | PASS | `/mnt/quant-lab-primary/research/reports/cr072-local-backtest-reports-20260616T081950/reports/`，772 files，4,529,789,958 bytes，SHA256 source/dest match |
| `data/` NAS 归档 | PASS | `/mnt/quant-lab-primary/data-lake/snapshots/cr072-local-backtest-data-20260616T081950/data/`，8 files，61,723,552 bytes，SHA256 source/dest match |
| 过程证据 NAS 归档 | PASS | `/mnt/quant-lab-primary/archive/migration-evidence/cr072-local-assets-evidence-20260616T081950/evidence-tree/`，2,091 files，47,503,432 bytes，SHA256 source/dest match |
| Manifest / checksum | PASS | `reports-*-sha256.txt`、`data-*-sha256.txt`、`evidence-*-sha256.txt` 已写入 NAS evidence root |
| 可重建缓存清理 | PASS | `.venv/`、`node_modules/`、`.pytest_cache/` 已删除；`__pycache__/` 数量从 132 降为 0 |
| 本地 `reports/` 保留 | PASS | 本地仍有 772 files，约 4.3G |
| 本地 `data/` 保留 | PASS | 本地仍有 8 files，约 59M |
| 远端 Git 写入 | PASS | 未执行 push；远端 `master` 仍为 `3ade165e8b1edad031a911490cf6c1cee942616e` |

## 剩余风险

- `reports/` 首次 `rsync -a` 在 NFS 上因 `chgrp Operation not permitted` 以 code 23 结束；随后使用 `--no-owner --no-group --no-perms --omit-dir-times` 重跑并以 code 0 完成，内容 SHA256 已确认一致。
- 过程证据首次复制时生成了一个平铺副本，已改名为 NAS 侧 `evidence-flat-initial-copy/`；正式审计入口为保留目录结构的 `evidence-tree/`。
- 本地 `reports/` / `data/` 未删除，若后续需要释放本地空间，必须单独授权删除门禁。

## 处理结论

- 审批结论：`closed-current-delivery`
- [x] 已获用户本轮授权。
- [x] 执行范围限定为 NAS 归档和可重建缓存清理。
- [x] 不删除本地 reports/data。

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | CR071 | Copy-first shadow root 已完成 CP7，尚未 CP8 关闭。 |
| NAS map | `docs/release/NAS-DIRECTORY-MAP-CR059.md` | 本轮 NAS 归档目标依据。 |
| Remote | `git@github.com:hyde-zhao/quant-lab.git` | 本轮只读检查，不写入。 |
