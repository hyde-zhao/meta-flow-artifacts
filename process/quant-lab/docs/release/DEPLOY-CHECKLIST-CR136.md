# Deploy Checklist CR136

## 适用性

| 项目 | 结论 | 说明 |
|---|---|---|
| 安装脚本 | N/A | CR136 不修改安装器。 |
| 外部部署 | N/A | 本地 offline runner 代码与测试变更。 |
| 数据迁移 | N/A | 不修改持久化状态或外部数据。 |
| 运行授权 | N/A | 不授权 runtime/NAS/QMT/credentials/provider/lake/catalog/trading。 |

## 验证入口

| 检查 | 命令 / 证据 | 结论 |
|---|---|---|
| runner regression | `pytest ... test_cr136 ... test_cr128 ...` | PASS, `42 passed` |
| hygiene | `scripts/check_process_artifact_hygiene.py --json` | PASS, `unclassified=0` |
| CR lifecycle | `meta-flow cr check` | PASS |
| State v2 | `meta-flow state check` | PASS |

## 发布前确认

- [ ] 用户接受 `READY_WITH_RISK`。
- [ ] 确认 CP8 approve 不授权真实发布、Git write、runtime 或交易。
