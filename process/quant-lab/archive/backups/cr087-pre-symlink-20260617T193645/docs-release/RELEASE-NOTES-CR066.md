---
status: ready
version: "1.0"
release_artifact_profile: compact
release_decision: READY_WITH_RISK
---

# CR066 Release Notes

## 1. 摘要

| 项目 | 内容 |
|---|---|
| 版本 | CR066 strategy-ready baseline |
| 发布结论 | READY_WITH_RISK |
| 发布范围 | Repository hygiene / clean publication strategy。 |
| 主要风险 | `R-CR066-001`: GitHub publication 尚未执行，需后续 CR067。 |

## 2. 版本号决策

| 项目 | 内容 |
|---|---|
| 当前版本 | N/A |
| 目标版本 | N/A |
| 变更类型 | rc |
| 兼容性 | compatible |
| 推荐原因 | 本轮只冻结策略和门禁，不发布运行包。 |

## 3. 新增能力 / 用户可见变化

| Change ID | 内容 | 影响用户 | 来源 |
|---|---|---|---|
| REL-CR066-001 | 明确当前分支不得 as-is publish。 | 避免发布 tracked forbidden history。 | CR062 preflight |
| REL-CR066-002 | 推荐 clean snapshot / allowlist manifest。 | 为后续 GitHub baseline 提供可审计路线。 | CR066 strategy |
| REL-CR066-003 | 将 CR067 设为后续执行门禁候选。 | 执行授权不会被 CR066 approve 隐含授予。 | DQ-CP5-CR066-05 |

## 4. 行为变化 / 修复问题

| Change ID | 类型 | 内容 | 用户影响 |
|---|---|---|---|
| REL-CR066-004 | behavior-change | CR062 blocker follow-up 从等待决策变为 CR066 strategy-ready。 | 后续应先走 CP8，再决定 CR067。 |

## 5. 破坏性变更

| Breaking ID | 是否存在 | 内容 | 迁移引用 |
|---|---|---|---|
| BR-CR066-001 | no | N/A | `docs/release/MIGRATION-CR066.md` |

## 6. 安装与升级

| 场景 | 方式 | 验证证据 |
|---|---|---|
| 安装 / 升级 / dry-run | N/A；CR066 不改变安装包。 | `docs/release/DEPLOY-CHECKLIST-CR066.md` |

## 7. 迁移说明

| 是否需要迁移 | 影响对象 | 说明 |
|---|---|---|
| no | N/A | `docs/release/MIGRATION-CR066.md` |

## 8. 已知问题与风险

| Risk ID | 严重度 | 状态 | 处理 |
|---|---|---|---|
| R-CR066-001 | HIGH | accepted-for-readiness | Publication execution deferred to CR067。 |
| R-CR066-002 | MEDIUM | open-for-CR067 | `.env.example` conditional include 仍需 future scan。 |
| R-CR066-003 | MEDIUM | open-for-CR067 | `reports/**` / `runs/**` 默认排除，若要发布需独立风险接受。 |

## 9. 回滚方式

| 回滚触发 | 回滚入口 | 说明 |
|---|---|---|
| CP8 reject 或用户取消 clean publication strategy | `docs/release/ROLLBACK-CR066.md` | 保持 CR062 blocked，不启动 CR067。 |

## 10. 参考链接

| 类型 | 路径 |
|---|---|
| Release Context | `process/release/RELEASE-CONTEXT-CR066.yaml` |
| CR066 Strategy | `docs/release/CLEAN-PUBLICATION-STRATEGY-CR066.md` |
| CR066 Manifest | `docs/release/CLEAN-PUBLICATION-MANIFEST-CR066.yaml` |
