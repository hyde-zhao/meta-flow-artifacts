# CR066 Tracked Forbidden Paths Audit

## 来源

来源文件：`process/checks/CR062-POST-APPROVAL-PREFLIGHT.md`

CR062 post-approval preflight 发现当前 Git index 已跟踪 CR062 exclude policy 中的路径。由于普通 push 会发布当前分支既有 tracked history，CR062 已 fail-closed。

## Findings

| Finding ID | 路径 | 当前事实 | 默认策略 | 风险 |
|---|---|---|---|---|
| CR066-AUDIT-001 | `reports/chapter3_factor_panel/**` | 已跟踪 | 不进入 clean public source baseline | 报告产物可能包含运行输出、真实数据衍生物或大文件。 |
| CR066-AUDIT-002 | `runs/RUN-EXEC-20260613-001.md` | 已跟踪 | 不进入 clean public source baseline | 运行态记录不应默认作为源码发布面。 |
| CR066-AUDIT-003 | `.env.example` | 已跟踪 | 可作为例外候选，前提是只含占位符且扫描通过 | 当前 CR062 `.env*` 模式未区分示例文件；需要显式 DQ。 |
| CR066-AUDIT-004 | `process/STATE.md` | 已跟踪且超过 1MB | 不阻断策略，但建议不进入精简 public baseline | 长期状态文件体积大、含历史内部路径和过程事实。 |
| CR066-AUDIT-005 | CR059 NAS 证据候选 | 未跟踪 / 候选证据含 NAS path / internal IP 字符串 | 默认不纳入 clean public baseline | 可能暴露内部拓扑；若纳入需 redaction 或风险接受。 |

## Recommended Classification

| 路径 / 模式 | 推荐分类 | 后续动作 |
|---|---|---|
| `reports/**` | exclude-from-public-baseline | clean manifest 排除；如必须发布，转 risk acceptance DQ。 |
| `runs/**` | exclude-from-public-baseline | clean manifest 排除；运行证据保留在本地审计。 |
| `.env.example` | conditional-allow | placeholder / secret scan PASS 后可纳入；否则排除。 |
| `process/STATE.md` | process-internal / heavy | 默认不纳入 public baseline；可保留在本地过程审计。 |
| `docs/release/NAS-*` / `ops/systemd/*.mount` | sensitive-topology-candidate | 默认排除或 redacted docs only。 |

## Stop Rule

任何后续 publication execution 在出现以下情况时必须 BLOCKED：

- clean manifest 含 `reports/**` 或 `runs/**` 且没有人工风险接受。
- `.env.example` 含真实 token、password、cookie、session、private key 或账户事实。
- manifest 含 NAS mount / internal topology 且没有 redaction 或风险接受。
- 需要 history rewrite、force push、branch rename 或 remote deletion 但没有独立 CR 授权。
