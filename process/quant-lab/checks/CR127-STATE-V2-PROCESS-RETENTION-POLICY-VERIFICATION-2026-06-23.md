# CR127 State v2 / Process Retention Policy Verification

## 结论

`CR-127 State v2 / Process Retention Policy Landing` 已完成并关闭，结论为 `READY`。

本次整改将默认机器状态入口切换为 `process/state/STATE.current.json`，将 `process/STATE.md` 降级为轻量人类摘要，并落地 process policies 与 read expansion 检查入口。旧 `STATE.md` 全文已归档为冷区审计材料。

## 变更对象

| 对象 | 状态 | 路径 |
|---|---|---|
| 轻量机器状态 | created | `process/state/STATE.current.json` |
| Base ledgers | created | `process/state/*-LEDGER.ndjson` |
| 真相源策略 | created | `process/policies/SOURCE-OF-TRUTH-MAP.yaml` |
| 保留策略 | created | `process/policies/RETENTION-POLICY.json` |
| 读取策略 | created | `process/policies/READ-POLICY.json` |
| 人类状态摘要 | compacted | `process/STATE.md` |
| legacy 全文归档 | archived | `process/archive/state/STATE.legacy-before-state-v2-2026-06-23.md` |
| CR context pack | created | `process/context/CP8-CR127-STATE-V2-PROCESS-RETENTION-CONTEXT.context.json` |
| CR summary | created | `process/changes/summaries/CR-127.summary.json` |
| CR evidence index | created | `process/archive/CR-127/evidence-index.json` |

## 检查结果

| 命令 | 结果 |
|---|---|
| `uv run --python 3.11 meta-flow workspace check` | PASS |
| `uv run --python 3.11 meta-flow state check --project-root /home/hyde/workspace/quant-lab` | PASS |
| `uv run --python 3.11 meta-flow check truth-map --project-root /home/hyde/workspace/quant-lab` | PASS |
| `uv run --python 3.11 meta-flow check retention-policy --project-root /home/hyde/workspace/quant-lab` | PASS |
| `uv run --python 3.11 meta-flow context check --context /home/hyde/workspace/quant-lab/process/context/CP8-CR127-STATE-V2-PROCESS-RETENTION-CONTEXT.context.json --project-root /home/hyde/workspace/quant-lab` | PASS |
| `uv run --python 3.11 meta-flow context read-log-check --project-root /home/hyde/workspace/quant-lab` | PASS |
| `uv run --python 3.11 meta-flow cr check --project-root /home/hyde/workspace/quant-lab` | PASS |
| `uv run --python 3.11 meta-flow check cr-tracking --project-root /home/hyde/workspace/quant-lab` | PASS |

## 关键观测

- `process/STATE.md` 已从约 2.1M 降为约 1.2K。
- `process/state/STATE.current.json` 是当前默认机器状态入口。
- `process/policies/READ-POLICY.json` 将 `process/STATE.md`、`process/DEVELOPMENT-PLAN.yaml`、完整 `process/changes/*.md`、完整 Story LLD 和 Story implementation 证据列入默认禁读。
- 需要读取 deny-default 文件时，必须通过 `process/state/READ-EXPANSION-LEDGER.ndjson` 记录原因；当前 read-log checker 通过。
- 当前没有 active formal CR；`CR-089` 仍为 blocked formal CR，既有 follow-up / spike candidates 仍是 backlog，不属于本 CR 的失败项。

## 不授权范围

本 CR 未授权以下动作，且验证期间未执行：

- 凭据、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志读取。
- NAS 访问、列取、挂载、读取、复制、写入、发布、pull 或删除。
- QMT / MiniQMT / XtQuant / gateway runtime 启动、连接、安装或运行。
- provider fetch、lake write、catalog publish 或 current pointer 修改。
- submit/cancel、buy/sell、simulation/live。
- 业务源码修改、runner 执行、`.gitignore` 修改、commit、push 或 remote write。

## 剩余事项

- 既有 `CR-089` 仍为 blocked formal CR。
- 既有 follow-up / spike candidates 仍按 `process/changes/CR-INDEX.yaml` 与 follow-up tracking 管理。
- 先前未启动的 runner implementation 不能再使用 `CR-127` 标签；若继续推进，应使用下一个可用 CR 编号。
