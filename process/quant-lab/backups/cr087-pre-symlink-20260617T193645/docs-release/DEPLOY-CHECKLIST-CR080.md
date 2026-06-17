---
status: ready
version: "1.0"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# Deploy Checklist CR080

## 1. 发布前输入检查

| 输入 | 状态 | 证据路径 | 说明 |
|---|---|---|---|
| Release Context Capsule | PASS | `process/release/RELEASE-CONTEXT-CR080.yaml` | READY_WITH_RISK。 |
| TEST-REPORT | PASS_WITH_RISK | `docs/quality/TEST-REPORT-CR080.md` | metadata-only risk accepted at CP8 pending。 |
| REVIEW | PASS_WITH_RISK | `docs/quality/REVIEW-CR080.md` | approve-with-risk。 |
| BLOCKER findings | 0 | `docs/quality/REVIEW-CR080.md` | 无。 |
| HIGH findings | 0 | `docs/quality/REVIEW-CR080.md` | 无。 |

## 2. 发布候选快照

| 检查项 | 状态 | 证据 / 摘要 |
|---|---|---|
| 变更范围清楚 | PASS | CR080 只覆盖 target root `reports/` / `data/` local restore/copy 和过程账本。 |
| 未跟踪文件已分类 | PASS | target `reports/` / `data/` 是本 CR 交付对象；过程文档是 release-content。 |
| 缓存与临时文件清理 | PASS | 本轮未生成 cache；不执行 cleanup。 |
| 敏感信息检查 | PASS_WITH_RISK | denylist count 0；不读取内容。 |

## 3. 安装 / 升级 / 幂等验证矩阵

| 平台 | 组件 | Scope | 场景 | 是否适用 | 验证命令 / 方法 | 结果 | N/A 原因 |
|---|---|---|---|---|---|---|---|
| Local FS | reports/data | target root | fresh restore/copy | yes | local `rsync -a --stats` with denylist excludes | PASS | N/A |
| Local FS | reports/data | target root | repeated dry-run / idempotency | yes | local `rsync -an --stats` | PASS | N/A |
| Codex / Claude | agents / skills / rules | project / user | install dry-run | no | N/A | N/A | 本轮不修改交付 agent/skill/rule 安装。 |
| All | uninstall / rollback | target local assets | destructive cleanup | no | N/A | N/A | 删除/移动 target assets 不授权。 |

## 4. 平台和权限边界

| Check ID | 检查项 | 状态 | 证据 / 说明 | 阻断等级 |
|---|---|---|---|---|
| DEP-CR080-001 | authoritative target root 正确 | PASS | `/home/hyde/workspace/quant-lab` | BLOCKING |
| DEP-CR080-002 | source root 正确 | PASS | `/home/hyde/workspace/local_backtest` retained assets | BLOCKING |
| DEP-CR080-003 | 不覆盖非空 target | PASS | preflight target missing | BLOCKING |
| DEP-CR080-004 | 不使用 `--delete` | PASS | execution deleted=0 | BLOCKING |
| DEP-CR080-005 | 回滚方案已确认 | PASS_WITH_RISK | `docs/release/ROLLBACK-CR080.md` | REQUIRED |

## 5. 发布结论

| 项目 | 内容 |
|---|---|
| release_artifact_profile | full |
| release_decision | READY_WITH_RISK |
| 阻断项 | 0 |
| 风险接受项 | R-CR080-001 / R-CR080-002 / R-CR080-003 |

## 6. 不授权项

| Item ID | 不授权操作 | 原因 | 需要的独立授权 |
|---|---|---|---|
| NA-CR080-001 | content read / parse / checksum | 本轮只授权 metadata-only verification | content validation CR |
| NA-CR080-002 | credentials / `.env` / account / trading facts | 敏感信息不在范围 | security/runtime gate |
| NA-CR080-003 | NAS/provider/lake/catalog/remote/runtime/old-root retirement | 外部、运行时或破坏性范围 | separate CR |
