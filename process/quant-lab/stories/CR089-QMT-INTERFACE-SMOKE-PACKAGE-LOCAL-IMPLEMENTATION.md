---
status: local-offline-package-ready
version: "0.1"
story_id: "CR089-S02"
story_slug: "qmt-interface-smoke-strategy-package"
feature_id: "CR089-QMT-INTERFACE-VALIDATION"
implementation_type: "mixed"
source_story: "process/changes/CR-089-QMT-INTERFACE-VALIDATION-GATE-2026-06-17.md"
source_design_evidence: "process/checkpoints/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md; process/checkpoints/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-REVIEW.md; process/checkpoints/CP5-CR089-QMT-INTERFACE-VALIDATION-READINESS.md"
created_by: "host-orchestrator"
created_at: "2026-06-17T22:03:03+08:00"
updated_at: "2026-06-17T22:03:03+08:00"
formal_cp6_claimed: false
runtime_authorized: false
---

# Implementation: CR089 qmt_interface_smoke Local Offline Package

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 在 CR089 已批准但未授权 runtime 的边界内，落地 `qmt_interface_smoke` 本地离线策略包骨架、manifest、checksum、人工 smoke guide、checker 和恢复上下文。 |
| 行为变化 | 仓库现在有可自动检查的 `packages/qmt_interface_smoke/0.1.0` 文件级包；交易主机可在本地缓存中做离线 intake 检查。 |
| 范围边界 | 不访问 NAS，不读取 `.env` / 凭据，不启动 QMT / MiniQMT / XtQuant / gateway，不查询账户，不 submit/cancel/simulation/live，不恢复 CR020，不激活 CR089，不推进 CR046。 |
| CP6 证据 | 本文件仅为本地离线实现证据；未声明 formal CP6 runtime PASS。 |

本次由 host-orchestrator / Codex 主线程在用户已同意的推荐方案边界内直接实现。未声称 meta-dev 子 agent 独立完成；当前平台工具约束下，用户没有明确要求 delegation，因此未主动 spawn 子 agent。

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| CR | `process/changes/CR-089-QMT-INTERFACE-VALIDATION-GATE-2026-06-17.md` | CR089 状态、允许动作、不授权项、策略包交付目标 |
| CP2 | `process/checkpoints/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md` | 范围限定：策略包 + 只读 smoke，不授权 runtime |
| CP3 | `process/checkpoints/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-REVIEW.md` | NAS 只作为 package exchange，交易主机使用本地不可变缓存 |
| CP5 | `process/checkpoints/CP5-CR089-QMT-INTERFACE-VALIDATION-READINESS.md` | CP5 推荐方案已 approved，但不授权 NAS / QMT / 凭据 / 账户 / 交易动作 |
| 状态 | `process/STATE.md` | active formal CR 仍为 CR046，CR089 保持 `blocked-readiness-approved` |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| process route health check | PASS | `meta-flow workspace check` 返回 `process_link_health: ok` |
| CR089 允许本地离线骨架 | PASS | CR089 frontmatter `approval_result=cp2-cp3-cp5-approved / no-runtime-authorization` |
| 待确认问题已关闭 | PASS_WITH_LIMITS | CR089 CP2/CP3/CP5 推荐项已 approved；runtime authorization 未批准，故不执行 runtime |
| 影响范围可定位 | PASS | 新增 `packages/qmt_interface_smoke/0.1.0`、`docs/qmt/CR089-...md`、checker、测试和本过程证据 |
| 验证方式明确 | PASS | 本地 checker、pytest、workspace check、CR tracking consistency、diff whitespace check |
| 当前 Wave / dev_gate 满足 | PASS_WITH_LIMITS | 仅限用户批准的本地离线 package skeleton；不声明 CR089 formal active / runtime CP6 |
| 文件所有权无冲突 | PASS | 新增独立 package / docs / script / test / process evidence 文件 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 是否必须 | 验证方式 |
|---|---|---|---|---|
| `packages/qmt_interface_smoke/0.1.0/manifest.yaml` | template-schema | 固化策略包 ID、target、只读接口合同、禁止动作和交付合同 | yes | checker + pytest |
| `packages/qmt_interface_smoke/0.1.0/checksums/SHA256SUMS` | template-schema | 固化关键文本文件 SHA256 | yes | checker + pytest |
| `packages/qmt_interface_smoke/0.1.0/**/README.md` | docs-handoff | 说明 package、QMT terminal、MiniQMT runner 的离线和后续运行边界 | yes | checksum + pytest |
| `packages/qmt_interface_smoke/0.1.0/validation/offline-intake-checklist.md` | docs-handoff | 给交易主机文件级 intake 使用 | yes | checksum + pytest |
| `packages/qmt_interface_smoke/0.1.0/evidence/redacted-smoke-result-template.yaml` | template-schema | 后续只读 smoke 脱敏证据模板 | yes | checker + pytest |
| `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | docs-handoff | 记录策略输出规范、交付位置合同和不授权项 | yes |人工审计 + diff check |
| `scripts/check_cr089_qmt_interface_smoke_package.py` | guardrail-test | 本地文件级 package checker | yes | CLI + pytest |
| `tests/test_cr089_qmt_interface_smoke_package.py` | guardrail-test | 回归保护 manifest、checksum、安全边界和 checker import 边界 | yes | pytest |
| `process/context/CP6-CR089-QMT-INTERFACE-SMOKE-PACKAGE-CONTEXT.yaml` | docs-handoff | 清上下文后的最小恢复入口 | yes | 人工审计 + workspace check |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| 不从 NAS 原地执行策略 | DQ-CP3-CR089-01 | `manifest.yaml`、CR089 plan、README | 写入 `direct_nas_execution_allowed=false` 和 package exchange 合同 | checker + pytest |
| 策略包使用 manifest + checksum + target docs + validation | DQ-CP3-CR089-02 | `packages/qmt_interface_smoke/0.1.0` | 新建 package skeleton | checker + pytest |
| 交易主机使用本地不可变缓存 | DQ-CP3-CR089-03 | `manifest.yaml`、CR089 plan | 写入 `${TRADING_PACKAGE_CACHE_ROOT}` 合同 | 文档审计 |
| 不允许交易写入能力 | DQ-CP3-CR089-05 / DQ-CP5-CR089-06 | manifest、evidence template、checker | 禁止 `submit_order`、`cancel_order`、simulation、live | pytest |
| 只读接口限定为 `query_positions` | DQ-CP2-CR089-02 / DQ-CP5-CR089-02 | `readonly_smoke_contract`、target docs、evidence template | 写入 endpoint、path、scope | checker + pytest |
| 不授权 agent 读取凭据或代跑 runtime | DQ-CR089-04 / DQ-CP2-CR089-04 | README、checklist、plan、context | 明确不读取 `.env` / 凭据，不启动 runtime | 文档审计 + import guard |

## 6. 单元测试 / Fixture 计划

| 测试对象 | 测试类型 | 输入 / Fixture | 期望 | 覆盖风险 | 状态 |
|---|---|---|---|---|---|
| package skeleton | structure-check | `packages/qmt_interface_smoke/0.1.0` | checker PASS | 文件缺失、manifest 漂移 | passed |
| manifest safety flags | contract | `manifest.yaml` | runtime / NAS / credential / account / trade flags 全为 false | 误授权 | passed |
| forbidden operation list | contract | `manifest.yaml` | 包含 NAS、凭据、QMT、账户原文、submit/cancel/simulation/live 禁止项 | 禁止项漏写 | passed |
| checksum drift | negative fixture | tmp copy + README 修改 | checker FAIL | 包内容被篡改 | passed |
| checker import boundary | AST | checker 源码 | 不导入网络、trading runtime、XtQuant、subprocess、dotenv | checker 越界 | passed |

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 输出文件 | 局部验证 | 状态 |
|---|---|---|---|---|---|
| IMPL-S1 | package manifest / target docs | package skeleton | `packages/qmt_interface_smoke/0.1.0/**` | `sha256sum` + checker | done |
| IMPL-S2 | guardrail / checksum | checker + tests | `scripts/check_cr089_qmt_interface_smoke_package.py`、`tests/test_cr089_qmt_interface_smoke_package.py` | `pytest` | done |
| IMPL-S3 | delivery plan | docs | `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | doc review + diff check | done |
| IMPL-S4 | context recovery | process evidence | 本文件 + context capsule | workspace check | done |

## 8. 变更说明

### 8.1 代码变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `scripts/check_cr089_qmt_interface_smoke_package.py` | create | 本地文件级 checker；不读 `.env`，不访问 NAS，不导入 trading runtime / QMT / XtQuant |

### 8.2 Prompt / Skill 变更

N/A。本次未修改 Agent / Skill。

### 8.3 模板 / Schema 变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `packages/qmt_interface_smoke/0.1.0/manifest.yaml` | create | 策略包 manifest |
| `packages/qmt_interface_smoke/0.1.0/checksums/SHA256SUMS` | create | 关键文本文件 SHA256 |
| `packages/qmt_interface_smoke/0.1.0/evidence/redacted-smoke-result-template.yaml` | create | 后续只读 smoke 脱敏结果模板 |

### 8.4 Guardrail / 测试变更

| 文件 / 命令 | 动作 | 说明 |
|---|---|---|
| `tests/test_cr089_qmt_interface_smoke_package.py` | create | 验证 package skeleton、禁止项、checksum drift 和 checker import 边界 |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr089_qmt_interface_smoke_package.py --package-root packages/qmt_interface_smoke/0.1.0` | run | PASS |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr089_qmt_interface_smoke_package.py` | run | PASS，5 passed |

### 8.5 文档变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `packages/qmt_interface_smoke/0.1.0/README.md` | create | package 说明、允许离线动作、不授权项和本地校验命令 |
| `packages/qmt_interface_smoke/0.1.0/targets/qmt_terminal/README.md` | create | QMT terminal target 边界 |
| `packages/qmt_interface_smoke/0.1.0/targets/miniqmt_runner/README.md` | create | MiniQMT runner target 边界 |
| `packages/qmt_interface_smoke/0.1.0/validation/offline-intake-checklist.md` | create | 交易主机 intake checklist |
| `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | create | 策略包输出规范、NAS exchange 合同、交易主机本地缓存合同 |
| `process/context/CP6-CR089-QMT-INTERFACE-SMOKE-PACKAGE-CONTEXT.yaml` | create | 清上下文恢复入口 |

## 9. 平台差异处理

| 平台 | 检查项 | 预期 | 结果 |
|---|---|---|---|
| Claude Code | direct ask agent 有 `AskUserQuestion` | N/A | 未修改 direct ask agent |
| Claude Code | non-direct agent 不声明 `AskUserQuestion` | N/A | 未修改 agent |
| Codex | 不写 Claude-only `tools` schema | yes | PASS |
| Codex | 无 `request_user_input` 时可降级 | N/A | 本次无人工选择 UI |
| install | dry-run 可执行 | N/A | 本次无安装器 |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr089_qmt_interface_smoke_package.py --package-root packages/qmt_interface_smoke/0.1.0` | PASS | `passed=true`，`errors=[]` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr089_qmt_interface_smoke_package.py` | PASS | `5 passed` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | PASS | `process_link_health: ok`；artifact repo dirty 为本轮过程文件变更 |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root /home/hyde/workspace/quant-lab` | PASS | `CR tracking consistency: PASS` |
| `git diff --check -- <touched files>` | PASS | 无输出 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| NAS publish / pull / list / copy | CR089 当前不授权 NAS 内容动作 | 单独 runtime / transfer authorization 后再执行 |
| QMT / MiniQMT / XtQuant / gateway 启动 | CR089 当前不授权 runtime | 单独 runtime authorization 后由用户手工执行 |
| `query_positions` 真实账户 smoke | 需要真实 QMT 环境、凭据和用户手工运行授权 | 使用 evidence template 回填脱敏摘要 |
| zip 构建 | 当前仅实现 skeleton，不构建二进制 / zip artifact | 发布前另行生成 zip 并更新 checksum / index |
| package index / approval / quarantine 文件真实写入 | NAS exchange 未授权 | 后续 publish gate 补齐 |

## 12. 风险与回滚

| Risk ID | 风险 | 影响 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|---|
| R-CR089-IMPL-001 | 把本地 skeleton 误认为 runtime-ready | 可能越权启动 QMT 或读取账户 | manifest、README、plan、context 全部写明 `runtime_authorized=false` | 发现误读时停止，回到 CR089 blocked-readiness-approved |
| R-CR089-IMPL-002 | 交易主机从 NAS 原地执行 | 网络抖动、半写入、审计不完整 | 规划和 manifest 要求本地不可变缓存 | 若必须临时原地读取，另起例外授权 |
| R-CR089-IMPL-003 | 脱敏证据模板被填入原文 | 泄露账户 / 交易信息 | 模板字段默认 raw output false，checklist 明确禁止 | 发现原文时不入库，重新脱敏 |

## 13. 设计缺口反馈

| Gap ID | 发现阶段 | 问题 | 应反馈到 | 是否阻塞 | 推荐处理 |
|---|---|---|---|---|---|
| GAP-CR089-IMPL-001 | implementation | CR089 批次内正式 Story 设计证据仍未完整输出，本次只覆盖用户批准的本地离线 skeleton 子集 | CR089 后续 LLD / CP5 batch | no | 保持状态为 `blocked-readiness-approved`，不要声称 formal CP6 / runtime PASS |
| GAP-CR089-IMPL-002 | implementation | 真实 package index / approval / quarantine 需要 NAS exchange 授权 | CR089 publish / transfer gate | no | 后续 runtime / transfer gate 再实现 |

## 14. QA / Review / Doc 后续交接

### QA 关注点

- checker 必须持续保持 file-only，不引入 `.env`、网络、trading runtime、QMT / MiniQMT / XtQuant 导入。
- checksum drift negative fixture 必须保持。
- manifest 中所有授权 flag 必须保持 false，除非出现独立授权 CR。

### Review 关注点

- 不要把 `docs/qmt/CR089-...PLAN.md` 中的 NAS 路径模板解释为已访问路径。
- 不要把 evidence template 的存在解释为真实 `query_positions` 已执行。
- 不要从旧根 `STATE.md` 或 CR020 恢复当前 QMT 验证状态。

### Doc 关注点

- 若后续生成 zip / package index / approval record，需要同步更新 manifest、checksums、CR089 plan 和本实现证据。
- 若用户清上下文后恢复，先读 `process/context/CP6-CR089-QMT-INTERFACE-SMOKE-PACKAGE-CONTEXT.yaml`。
