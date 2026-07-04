---
status: ready
version: "1.0"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# Release Notes CR080

## 1. 摘要

| 项目 | 内容 |
|---|---|
| 版本 | process-only-cr080 |
| 发布结论 | READY_WITH_RISK |
| 发布范围 | 将 legacy retained assets 中的 `reports/` 与 `data/` 受控复制到 authoritative target root。 |
| 主要风险 | R-CR080-001 metadata-only 验证不证明内容 hash / 内容质量。 |

## 2. 版本号决策

| 项目 | 内容 |
|---|---|
| 当前版本 | process-only |
| 目标版本 | process-only-cr080 |
| 变更类型 | PATCH |
| 兼容性 | compatible |
| 推荐原因 | 不改代码 API、包名或配置 schema；仅补齐 target root 本地资产和过程账本。 |

## 3. 新增能力 / 用户可见变化

| Change ID | 内容 | 影响用户 | 来源 |
|---|---|---|---|
| REL-CR080-001 | `/home/hyde/workspace/quant-lab/reports/` 已存在，并与 legacy source 在 metadata quick-check 层面对齐。 | 使用 target root 的日常工作流可见 reports 目录。 | CR-080 |
| REL-CR080-002 | `/home/hyde/workspace/quant-lab/data/` 已存在，并与 legacy source 在 metadata quick-check 层面对齐。 | 使用 target root 的日常工作流可见 data 目录。 | CR-080 |

## 4. 行为变化 / 修复问题

| Change ID | 类型 | 内容 | 用户影响 |
|---|---|---|---|
| REL-CR080-003 | fix | 修复 CR079 发现的 target root 缺少 `reports/` / `data/` 问题。 | target root 本地资产完整性提升。 |
| REL-CR080-004 | process-ledger | STATE、CR-INDEX、CR080 CR 文件已从 preflight pending 收敛为 copy executed。 | 后续 CR 决策可基于当前事实。 |

## 5. 破坏性变更

| Breaking ID | 是否存在 | 内容 | 迁移引用 |
|---|---|---|---|
| BR-CR080-001 | no | 无代码、schema、配置或命令参数破坏性变更。 | `docs/release/MIGRATION-CR080.md` |

## 6. 安装与升级

| 场景 | 方式 | 验证证据 |
|---|---|---|
| target root local asset restore | local `rsync -a --stats` with denylist excludes | `process/checks/CR080-DATA-REPORTS-RESTORE-EXECUTION-2026-06-17.md` |
| post-copy quick-check | metadata-only `find` / `du`; `rsync -an --stats` | `process/checks/CP7-CR080-DATA-REPORTS-RESTORE-VERIFICATION-DONE.md` |

## 7. 迁移说明

| 是否需要迁移 | 影响对象 | 说明 |
|---|---|---|
| yes | target root `reports/` / `data/` local assets | 已完成一次 legacy -> target copy；不涉及 schema / config migration。 |

## 8. 已知问题与风险

| Risk ID | 严重度 | 状态 | 处理 |
|---|---|---|---|
| R-CR080-001 | MEDIUM | pending CP8 acceptance | metadata-only 不证明内容 hash / 内容质量。 |
| R-CR080-002 | LOW | pending CP8 acceptance | NAS/provider/lake/catalog 未验证。 |
| R-CR080-003 | LOW | pending CP8 acceptance | 旧根仍保留，未退役。 |

## 9. 回滚方式

| 回滚触发 | 回滚入口 | 说明 |
|---|---|---|
| 用户认为 metadata-only 风险不可接受 | `docs/release/ROLLBACK-CR080.md` | 默认只回滚 process ledger；目标目录删除 / 移动需要独立 destructive authorization。 |

## 10. 参考链接

| 类型 | 路径 |
|---|---|
| Release Context | `process/release/RELEASE-CONTEXT-CR080.yaml` |
| CP6 | `process/checks/CP6-CR080-DATA-REPORTS-RESTORE-CODING-DONE.md` |
| CP7 | `process/checks/CP7-CR080-DATA-REPORTS-RESTORE-VERIFICATION-DONE.md` |
| Test Report | `docs/quality/TEST-REPORT-CR080.md` |
| Review | `docs/quality/REVIEW-CR080.md` |
