---
context_id: "QMT-INTERFACE-VALIDATION-CR-CONTEXT-2026-06-17"
status: "ready"
created_at: "2026-06-17T20:10:23+08:00"
owner: "host-orchestrator"
source: "user requested QMT interface validation CR整理"
active_change: "CR-046"
runtime_authorization_granted: false
credential_read_authorized: false
qmt_runtime_authorized: false
miniqmt_runtime_authorized: false
---

# QMT 接口验证相关 CR 上下文

## 整理结论

当前可恢复的 QMT 相关主线不是 CR020，而是 `CR-046`。

`CR-020` 已在 2026-06-10 因用户确认无法获取 MiniQMT 权限并要求删除 QMT 路线，被标记为 `deleted-by-user`。它的历史 CP2-CP7 证据只能用于审计，不能作为本次 QMT 接口验证入口。

## 当前可恢复主线

| CR | 状态 | 当前门禁 | 适用范围 | 明确不授权 |
|---|---|---|---|---|
| `CR-046` | `active-cp6-pass-ready-for-verification`，用户于 2026-06-14 挂起 | 恢复后从 CP7 verification-execution 继续 | QMT / MiniQMT 双目标策略交付框架、验证框架、MiniQMT runner 安装设计、策略包契约的静态与 fixture 验证 | 真实传输、真实导入、QMT 终端运行验证、MiniQMT 连接、凭据读取、账户 / 资金 / 持仓 / 委托 / 成交查询、submit/cancel、simulation/live |

CR046 的 CP6 证据已经 PASS：`process/checks/CP6-CR046-DUAL-TARGET-FRAMEWORK-BATCH-A-CODING-DONE.md`。目前没有 `CP7-CR046` 产物，恢复后应先做 CR046 框架层 CP7，而不是直接连 QMT。

## 已删除或不可直接恢复

| CR | 状态 | 处理结论 |
|---|---|---|
| `CR-020` | `deleted-by-user` | 不再等待 MiniQMT，不执行 Windows/QMT gateway 实机验证；未来恢复 QMT 必须重新发起 CR。 |
| `CR-021` | `cancelled-user-deleted` | 不启动 QMT simulation。 |
| `CR-022` | `cancelled-user-deleted` | 不启动 live-readonly。 |
| `CR-023` | `cancelled-user-deleted` | 不启动 small-live。 |
| `CR-024` | `cancelled-user-deleted` | 不启动 scale-up。 |

## 历史上下文 CR

| CR | 状态 | 对本次验证的意义 |
|---|---|---|
| `CR-019` | closed | 早期 Stage6 QMT FastAPI 本地服务桥接架构已关闭；后续真实 QMT 运行必须独立 CR。 |
| `CR-040` | closed-current-delivery | 记录 2026-06-10 删除 QMT 路线并转向 Paper Simulation / Goldminer 候选路线的历史决策。 |
| `CR-045` | closed-current-delivery | Goldminer Windows bridge readonly skeleton 已关闭；不授权 Goldminer runtime，也不替代 QMT 验证。 |
| `CR-079` / `CR-080` | closed-current-delivery | data/reports 访问与恢复已关闭；不授权 QMT/MiniQMT runtime 或 CR046 recovery。 |
| `CR-087` / `CR-088` | closed-local-remediation | 过程文档归档与 artifact root 已纠偏到 `/home/hyde/workspace/meta-flow-artifacts/process/quant-lab`；只影响文档路由，不授权 runtime。 |

## 后续启动路径

### 路径 A：先恢复 CR046 CP7

适用条件：只是要把 QMT / MiniQMT 双目标框架的静态、schema、fixture 和不授权边界验证收敛。

推荐动作：

1. 用户明确恢复 CR046。
2. 生成或刷新 CP7 context。
3. 执行 CR046 framework-first verification。
4. 若 CP7 通过，再准备 CP8 人工终验。

该路径仍不得连接 QMT，不得读取 `.env`，不得查询账户，也不得执行 submit/cancel。

### 路径 B：真实 QMT 接口验证新 CR

适用条件：用户要真正验证 QMT 接口可用性，例如登录、会话、只读查询、终端导入、shadow run、submit/cancel 或 MiniQMT / XtQuant 连接。

推荐新建独立 CR，例如 `CR-089-QMT-INTERFACE-VALIDATION-GATE`，并在 CP2 先确认：

| 决策面 | 必须明确 |
|---|---|
| 验证目标 | 只读登录 / session、capabilities、query positions、QMT terminal shadow、MiniQMT readonly、submit/cancel 中具体哪一项 |
| 运行环境 | Windows QMT 机器、QMT 终端状态、是否涉及 WSL / bridge / runner |
| 权限边界 | 是否允许读取 `.env` 或凭据，是否允许读取账号 ID，是否允许账户 / 资金 / 持仓 / 委托 / 成交查询 |
| 安全边界 | redaction、日志脱敏、kill switch、失败即停止、证据落盘范围 |
| 交易边界 | 是否完全禁止 submit/cancel；若不禁止，必须另起更高风险门禁和逐 run 授权 |
| 回滚与停止 | 停止命令、端口 / 进程清理、证据保留、不得自动重试的条件 |

## 当前不授权项

- 不读取 `.env`、token、交易密码、session 或任何凭据。
- 不启动 QMT gateway、Windows bridge、MiniQMT runner 或任何交易相关服务。
- 不连接 QMT / MiniQMT / XtQuant。
- 不查询真实账户、资金、持仓、委托、成交。
- 不 submit/cancel。
- 不做 simulation/live。
- 不做 provider fetch、lake write、catalog publish。
- 不把 CR020 的历史 fixture/static PASS 解释为当前 QMT 接口可用。

## 核心证据路径

- `process/changes/CR-INDEX.yaml`
- `process/changes/CR-046-TERMINAL-NATIVE-SIMULATION-STRATEGY-EXPORT-2026-06-13.md`
- `process/changes/CR-046-FOLLOW-UP-TRACKING-2026-06-13.md`
- `process/checks/CP6-CR046-DUAL-TARGET-FRAMEWORK-BATCH-A-CODING-DONE.md`
- `docs/qmt/CR046-DUAL-TARGET-STRATEGY-FRAMEWORK.md`
- `docs/qmt/CR046-VERIFICATION-FRAMEWORK.md`
- `docs/qmt/CR046-MINIQMT-RUNNER-INSTALL-DESIGN.md`
- `process/changes/CR-020-QMT-WINDOWS-GATEWAY-SERVER-LOGIN-READONLY-QUERY-ADMISSION-2026-06-04.md`
- `process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md`
