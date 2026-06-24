# Release Notes CR137

## 结论

| 字段 | 内容 |
|---|---|
| release_decision | `READY_WITH_RISK` |
| release_artifact_profile | `compact` |
| 范围 | Offline runner run registry |
| 风险接受 | `WARN-CR137-CR-TRACKING` |

## 用户可见变化

- 新增 `RunRegistryEntry` 和本地 `runner-runs.index.json` registry 合同。
- successful validated bundle run 可追加 pass registry entry。
- 已通过 spec 校验但执行 blocked 的 run 可追加 diagnostic entry，但不得链接 pass bundle。
- `RunSpec` 新增可选 `run_registry_output_path`；使用 registry output 时必须同时提供 bundle output。
- CLI 新增 `--registry-output`、`--append-registry-bundle`、`--list-registry`、`--inspect-registry --run-id`。
- registry 路径和 bundle 路径包含 `.env`、`token`、`secret`、`credential`、`credentials` 时 fail closed。

## 兼容性

- 既有 RunSpec、RunResult、evidence index 和 artifact bundle 合同保持兼容。
- 新 registry 能力是可选 additive 行为。
- registry 只保存轻量索引；完整可 inspect/replay 对象仍是 validated artifact bundle。

## 不授权项

本 CR 不授权 runtime/NAS/QMT/credentials/provider/lake/catalog/trading/Git write。
