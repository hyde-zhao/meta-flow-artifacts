# Release Notes CR136

## 结论

| 字段 | 内容 |
|---|---|
| release_decision | `READY_WITH_RISK` |
| release_artifact_profile | `compact` |
| 范围 | Offline runner bundle schema / compatibility validation |
| 风险接受 | `WARN-CR136-CR-TRACKING` |

## 用户可见变化

- 新增 `validate_run_artifact_bundle()` public API。
- `inspect_run_artifact_bundle()` 和 `replay_run_artifact_bundle()` 在读取前会先执行 bundle validation。
- CLI 新增 `--validate-bundle`，valid bundle 输出 manifest，坏 bundle 返回 blocked JSON / 非零退出。
- 缺文件、坏 JSON、不兼容 schema、hash/size 不一致、状态不一致或授权边界异常的 bundle 会 fail closed。

## 兼容性

- valid CR135 bundle 文件结构保持兼容。
- 过去可能被 inspect/replay 读取的损坏 bundle 现在会被拒绝，这是预期的安全收紧。

## 不授权项

本 CR 不授权 runtime/NAS/QMT/credentials/provider/lake/catalog/trading/Git write。
