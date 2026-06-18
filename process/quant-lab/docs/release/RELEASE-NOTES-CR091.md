# CR091 Release Notes

> release_decision: `READY_WITH_RISK`
> profile: `compact`
> generated_at: `2026-06-18T15:24:12+08:00`

## 范围

CR091 交付 QMT strategy runner 的离线合同与最小实现切片。当前交付包含：

| 类型 | 内容 |
|---|---|
| 研究 / 设计 | runner reference research、HLD、LLD、TEST-PLAN |
| 代码 | `trading/strategy_runner/` thin module |
| 策略能力 | 多因子优先 adapter，并保留 legacy / package adapter 扩展合同 |
| 包处理 | 本地 package manifest / checksum / immutable cache / active pointer |
| 网关边界 | fake readonly gateway wrapper，仅覆盖 health / capabilities / query_positions 合同 |
| 证据 | redacted evidence summary 与 forbidden operation counters |
| 验证 | offline checker、fixtures、contract tests、CP6 / CP7 证据 |

## 用户可见变化

- 可以在离线 fixture 范围验证策略包 intake、adapter 输出和只读 runner 合同。
- 多因子策略是首选输入形态，其他策略通过统一 `StrategyAdapter` 合同扩展。
- 当前 runner 输出 `TargetPortfolioSnapshot` 和 `OrderIntentDraftV1`，但这些输出明确带有 `qmt_allowed=false` 和 `not_authorization=true` 语义。
- evidence 只允许脱敏摘要；禁止输出账户、持仓、资金、委托、成交或日志原文。

## 质量结论

| 检查 | 结果 |
|---|---|
| CP6 implementation | `PASS` |
| CP7 verification | `PASS_WITH_RISK` |
| CR091 pytest | `13 passed` |
| offline checker | `passed=true`，forbidden counters 全 0 |
| static boundary scan | PASS |
| py_compile | PASS |
| review findings | none-found |

## 已知风险

| 风险 ID | 等级 | 说明 |
|---|---|---|
| `R-CR091-CP7-001` | HIGH | 离线 CP7 不证明真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 可用，也不证明策略可交易。 |
| `R-CR091-CP7-002` | MEDIUM | `meta-flow check cr-tracking` 仍因历史 CR025 / CR019 台账旧账返回非零；当前 active formal CR 为 CR091。 |
| `R-CR091-CP7-003` | LOW | `write_evidence_summary()` 支持本地写文件，未来启用时需要输出目录和敏感信息扫描门禁。 |

## 不授权项

CR091 CP8 即使通过，也不授权：

- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文读取。
- submit / cancel、buy / sell。
- simulation / live。
- provider / lake / catalog / publish。
- CR089 自动启动或 CR020 gateway 路线恢复。

## 版本判断

| 字段 | 内容 |
|---|---|
| 当前版本 | `0.0.0-local` |
| 目标内部版本 | `0.1.0-cr091-offline-runner` |
| SemVer 判断 | internal minor |
| 原因 | 新增 runner 离线模块和合同，但不发布公开 release，不执行真实运行。 |

## 回滚摘要

若 CP8 被 reject 或后续发现离线合同不安全，回滚目标为 `CR091 CP7 PASS_WITH_RISK pending CP8`。回滚只涉及代码 / 测试 / 过程账本状态，不需要删除真实数据，不需要访问 NAS，也不需要启动 runtime。
