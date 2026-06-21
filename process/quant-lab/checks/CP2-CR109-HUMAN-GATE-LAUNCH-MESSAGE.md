---
check_id: "CP2-CR109-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-21T22:26:34+08:00"
checkpoint: "process/checkpoints/CP2-CR109-DOCS-FUNCTIONAL-ORG-REVIEW.md"
---

# CP2 CR109 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR109-DOCS-FUNCTIONAL-ORG-REVIEW.md`

自动预检结论：PASS_WITH_RISK，阻断项 0。

Context Capsule：`process/context/CP2-CR109-DOCS-FUNCTIONAL-ORG-CONTEXT.yaml`，状态 ready-for-scope-review，read_profile=compact。

决策收集覆盖：已扫描用户纠偏、CR109 design、现有 docs/design/docs/features/docs/quality；候选问题 7 项，纳入本门待决策 4 项；N/A 来源原因见 checkpoint。

本轮待人工决策项：4。

如果你回复 approve，表示你接受以下 4 项推荐方案，不表示授权下方不授权项。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR109-01 | scope | 是否由 CR109 承接 CR108 未完成的 docs 功能化缺口？ | 确认，由 CR109 承接。 | 重开 CR108；只记录 follow-up。 | 新 CR 保持历史清晰。 | CR tracking hygiene 顺延。 |
| DQ-CP2-CR109-02 | implementation | 是否允许物理移动 CR-scoped docs 到功能域 evidence/by-cr？ | 允许，先生成 migration map，再移动。 | 只新增功能索引；只移动 docs/features。 | 推荐方案真正解决目录按 CR 组织。 | 旧路径引用需迁移映射。 |
| DQ-CP2-CR109-03 | implementation | 是否允许 docs/quality 按 by-feature/by-cr 重组？ | 允许，保留历史报告正文。 | 暂缓 quality。 | 推荐方案解决质量证据平铺。 | 文件移动量较大。 |
| DQ-CP2-CR109-04 | risk_acceptance | 是否接受移动路径带来的旧引用风险？ | 接受，要求 migration map 和静态引用扫描。 | 不接受，只做索引。 | 推荐方案可达成功能目录目标。 | 可能出现 broken refs。 |

不授权项：

- 不授权读取凭据、`.env`、token、API key、cookie、私钥、账号。
- 不授权 runtime、SaaS、trace upload、production write、真实交易或真实发布。
- 不授权启动 CR-033。
- 不授权关闭或处理 CR102-CR104。
- 不授权业务代码整改。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
