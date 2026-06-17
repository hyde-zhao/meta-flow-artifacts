---
status: ready
version: "1.0"
release_artifact_profile: compact
release_decision: READY_WITH_RISK
---

# CR066 Deploy Checklist

## 1. 发布前输入检查

| 输入 | 状态 | 证据路径 | 说明 |
|---|---|---|---|
| Release Context Capsule | PASS | `process/release/RELEASE-CONTEXT-CR066.yaml` | CR066 scoped release context 已生成。 |
| TEST-REPORT | N/A | N/A | CR066 无代码实现和 runtime verification。 |
| REVIEW | N/A | N/A | 本轮只做策略门禁；无实现 diff review。 |
| BLOCKER findings | 0 | CP8 checklist | 无阻断项；剩余风险进入 CP8 DQ。 |
| HIGH findings | accepted | `DQ-CP8-CR066-02` | Publication 延迟和 CR067 前置为已知风险。 |

## 2. 发布候选快照

| 检查项 | 状态 | 证据 / 摘要 |
|---|---|---|
| 变更范围清楚 | PASS | CR066 scope / audit / strategy / manifest / checkpoint。 |
| 未跟踪文件已分类 | PASS_WITH_RISK | CR066 产物为未跟踪 process/docs 文件，尚未 Git stage。 |
| 缓存与临时文件清理 | N/A | 未执行构建或测试缓存生成。 |
| 敏感信息检查 | PASS_WITH_RISK | 未读取凭据；后续 CR067 仍必须执行 no-secret / topology scan。 |

## 3. 安装 / 升级 / 幂等验证矩阵

| 平台 | 组件 | Scope | 场景 | 是否适用 | 验证命令 / 方法 | 结果 | N/A 原因 |
|---|---|---|---|---|---|---|---|
| Git repository | publication strategy | project | readiness document review | yes | checkpoint + release context review | PASS | N/A |
| Codex / Claude | agents / skills / rules | project / user | install / upgrade / dry-run | no | N/A | N/A | CR066 不改变安装器或平台交付包。 |
| All | uninstall / rollback | project / user | rollback | yes | `docs/release/ROLLBACK-CR066.md` | PASS_WITH_RISK | 回滚为状态分流，不执行文件删除。 |

## 4. 平台和权限边界

| Check ID | 检查项 | 状态 | 证据 / 说明 | 阻断等级 |
|---|---|---|---|---|
| DEP-CR066-001 | 平台路径符合 contract | N/A | 不涉及平台安装路径。 | N/A |
| DEP-CR066-002 | Claude direct ask tools 权限正确 | N/A | 不涉及 Claude subagent 文件。 | N/A |
| DEP-CR066-003 | Codex 不包含 Claude-only schema | N/A | 不涉及 Codex agent / skill schema。 | N/A |
| DEP-CR066-004 | 不覆盖用户本地配置 | PASS | 未修改 `.env`、Git config 或本地 runtime config。 | REQUIRED |
| DEP-CR066-005 | 回滚方案已确认 | PASS | `docs/release/ROLLBACK-CR066.md`。 | REQUIRED |

## 5. 发布结论

| 项目 | 内容 |
|---|---|
| release_artifact_profile | compact |
| release_decision | READY_WITH_RISK |
| 阻断项 | 0 |
| 风险接受项 | `DQ-CP8-CR066-02`, `DQ-CP8-CR066-04` |

## 6. 不授权项

| Item ID | 不授权操作 | 原因 | 需要的独立授权 |
|---|---|---|---|
| NA-CR066-001 | Git add / commit / remote add / push | CR066 strategy-only。 | CR067 CP5 runtime_authorization |
| NA-CR066-002 | Branch creation / rename / history rewrite / force push | 高风险 Git governance。 | 独立 CR |
| NA-CR066-003 | NAS / lake / provider / runtime / credential | 不在 CR066 范围。 | 独立 CR |
