---
checkpoint_id: "CP7"
checkpoint_name: "验证完成检查"
type: "auto"
status: "PASS_WITH_RISK"
story_id: "STORY-038-05"
story_slug: "pppoe-client-op"
cr_id: "CR-038"
wave: 2
design_evidence_type: "full-lld"
validation_mode: "mixed"
executed_by: "meta-qa"
executed_at: "2026-08-15"
---

# CP7 验证完成检查 — STORY-038-05 PPPoE Client 拨号 op

## Entry Criteria

| 条目 | 结果 |
|---|---|
| Story `status=ready-for-verification` | PASS |
| CP6 编码完成门 PASS | PASS |
| full-lld 设计证据已确认 | PASS（LLD 14 章节 + IMPLEMENTATION.md） |
| validation_mode 判定（mixed，真机拨号 N/A） | PASS |

## 验证对象清单

| 对象 | 文件 | 核对结果 |
|------|------|---------|
| `INTENT_TO_OP`/`UNAVAILABLE_OPS`/`PPPOE_CLIENT_MANUAL_STEPS`/`_CIPHER_PREFIXES`/`_SECRET_KEYS` | op_mapper.py | 与 LLD §3.3/3.4/3.5/§10 一致 |
| `_probe_op_availability`/`_assert_no_plaintext_secret`/`_build_manual_steps`/`map_intent` | op_mapper.py | 与 LLD §5/§6 一致 |
| `tests/test_op_mapper_pppoe_client.py`（17 例） | 根 tests/ | 全绿 |

## 验证追踪矩阵

| 场景 | 结果 |
|------|------|
| SCN-038-POS-005（op 映射） | PASS（TC-POS-005-01/02） |
| SCN-038-PRE-001（op 预检） | PASS（TC-PRE-001-01/02） |
| RA-038-001 闭环 | PASS（TC-RA-001-01/02） |
| DQ-038-03 三选一 | PASS（TC-ABORT-01 + 非法 fallback ValueError） |
| ADR-02 凭据占位 | PASS（TC-SEC-01 4 例） |
| NFR-3 向后兼容 | PASS（validate_mapping_consistency + EXPECTED_OP_COUNT=26） |

## 设计契约验证

- `map_intent` 签名/返回结构与 LLD §3.1/3.2 一致；三选一默认 atomic-skip 写 reason，abort 抛 OpNotFoundError，非法 fallback 抛 ValueError（禁止静默跳过）。
- `UNAVAILABLE_OPS` 快照字段齐全，`external_dep_ref` 固化跨仓库依赖。
- §10.1 cipher 表述歧义按 LCQ-STORY-038-03-01 方案 A 收敛为保守密文前缀 allowlist（`$c$3$`/`%^%#`/`$1$`/`$5$`/`$6$`），与 S03 一致。

## 分层验证结果

- 单测：`tests/test_op_mapper_pppoe_client.py` 17/17 PASS。
- 回归：`tests/test_op_mapper_rollback.py tests/test_step_refs.py tests/test_tg_op_mapping.py` 76 PASS。
- 手动 dry-run：`map_intent("pppoe_client", {username, password="${ENV_PPPOE_USER_PASSWORD}"})` → available=False + fallback=atomic-skip + reason + 6 条 manual_steps，无明文。
- 真机拨号：N/A（runtime_authorization，未执行）。

## 功能 / 异常 / 回归 / 集成

- 功能：op 映射 + 三选一确定性分发闭环。
- 异常：intent 未识别抛 OpNotFoundError；明文凭据抛 ValidationError；abort 抛 OpNotFoundError；非法 fallback 抛 ValueError。
- 回归：现有 26 op 四表不变（EXPECTED_OP_COUNT=26）。
- 集成：编排层 `manual_steps` 消费分支属调用方（S11 ptm-te / case_runner），本 Story 边界外。

## 非功能

- 确定性（静态表+快照，无 subprocess/网络）、无副作用（纯函数）、向后兼容（26 op 不变）、性能 O(1)。

## 缺陷 / 问题

| 项 | 说明 |
|----|------|
| Return Packet + Evidence Index 缺失 | 机器真相源缺位，用 IMPLEMENTATION.md + CP6 检查 + LLD 作为等价证据（非实现缺陷，建议补记） |
| cipher 设计缺口（LLD §10.1 表述歧义） | 实现按方案 A 收敛，已留痕，无需回修 |

## 剩余风险

- DUT PPPoE Client 真机拨号（获 163.0.0.x）N/A，独立 runtime_authorization。
- ptm-atomic op 扩展（OPEN-038-05-01）follow-up，重访条件：ptm-atomic 升级或 CP8 复核。

## 阶段决策

**PASS_WITH_RISK** —— 实现与 full-lld 契约闭环，测试全绿，无明文凭据；真机拨号 N/A + 外部依赖 follow-up 汇入 CP8。路由：host-orchestrator。
