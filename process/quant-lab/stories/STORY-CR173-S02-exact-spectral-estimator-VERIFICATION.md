# CR-173 S02 CP7 独立验证报告

- Story：`CR173-S02-exact-spectral-estimator`
- 验证阶段：CP7
- 验证角色：`meta-qa-critical`
- validation_mode：`mixed`（repository-local Python 执行 + 静态/语义审查）
- context：`process/context/stories/STORY-CR173-S02.CP7.verify-packet.json`
- 结论：`PASS`
- 检查时间：`2026-07-17T11:26:07+08:00`

## 1. 验证范围与非范围

本轮验证 S02 的 exact-rational 输入域、确定性 fraction-free `LDLᵀ`、参与率公式、一次 half-even 舍入、双范围不变量、method/hash 绑定以及 fixture-only 授权边界。S01 合同测试仅作为上游回归输入。

本轮不修改 `engine/**`、`tests/**` 或 `docs/**`，不引入矩阵规模上限、浮点容差、clamp、随机主元或 production SLA，不接入 public C1、真实数据、runtime、credential、provider、交易或 Git remote。

## 2. 前置条件

| 条件 | 结果 | 证据 |
|---|---|---|
| process route health | PASS | `STATE.current.json` 与 `CURRENT.json` 均指向本 S02 CP7 packet，`health=ok` |
| Story 已完成 CP6 | PASS | `CP6-CR173-S02-IMPLEMENTATION-DONE.result.json`：`decision=PASS` |
| 独立 QA dispatch 存在 | PASS | `AD-CR173-S02-CP7-META-QA-CRITICAL-20260717T112048+0800` |
| 验证环境可用 | PASS | `uv run --python 3.11`、pytest、py_compile 与 Git 静态检查可执行 |
| 授权边界 | PASS | fixture/static only；deny-default 操作未发生 |

## 3. 验证对象清单

| 对象 | 验证方式 | 结果 |
|---|---|---|
| `engine/effective_trial_estimator.py` | 全文静态审查、pytest、py_compile、exact oracle 对抗扫描 | PASS |
| `engine/effective_trial_evidence.py` | 上游 method/hash/evidence trust binding 回归 | PASS |
| `tests/research/test_effective_trial_estimator.py` | 27 个数值/边界测试执行与覆盖审查 | PASS |
| `tests/research/test_effective_trial_evidence_contracts.py` | 60 个上游合同与 adversarial regression | PASS |
| CP6 return/evidence/result | 证据完整性、风险与授权声明复核 | PASS |
| fixture-only 性能边界 | 检查实现未新增 cap/优化承诺/SLA，保留已接受风险 | PASS |

## 4. 验证追踪矩阵

| Requirement / Scenario | 设计合同 | 实现对象 | 验证证据 | 结果 | 风险 |
|---|---|---|---|---|---|
| REQ-001 / SC-CR173-P01 | `n² / ΣRij²` 的二阶有效维度 | `estimate_participation_ratio_exact` | analytic oracle 4/4；exact equality | PASS | SECOND-ORDER-MODEL-BIAS 保持 claim ceiling |
| REQ-002 / SC-CR173-Q01 | declared_exact、canonical base-10、F03/F04 互斥 | normalization + matrix validator | invalid-token 9 类、finite-domain 5 类、静态 float bridge=0 | PASS | 无新增风险 |
| REQ-003 / SC-CR173-F01 | 失败 fail-closed | facade + failure basis | F03/F04/F08 targeted regression | PASS | 无 |
| REQ-004 / SC-CR173-N01 | 七字段 standalone evidence 与 trust binding | evidence builder | S01 60/60 回归、method/hash 绑定复核 | PASS | public projection 仍在范围外 |
| REQ-005 / SC-CR173-B01 | strategy-agnostic / fixture-only | module import/lexeme scan | public/strategy/real/runtime dependency 0 | PASS | 无 |
| REQ-006 / SC-CR173-D01 | 确定性 result/computation/evidence identity | exact estimator + canonical hash | 3/3 repeat 产生 1/1/1 identity | PASS | 无 |
| REQ-007 / SC-CR173-C01 | standalone only，不触碰 public C1 | imports/calls/static boundary | public dependency edge/call/write=0 | PASS | projection 延后到独立 CR |
| REQ-008 / SC-CR173-A01 | deny-default authorization | source/static/runtime observation | credential/real/runtime/remote operation=0 | PASS | 无 |

## 5. 设计契约验证

| 合同 | 验证方式 | 阻塞性 | 结果 |
|---|---|---:|---|
| 仅 canonical base-10 token 进入 exact 域 | parser tests + source review | 是 | PASS |
| F03 表示 representation，F04 仅表示 finite exact domain | targeted classification tests | 是 | PASS；overlap=0 |
| PSD 判定不使用容差 | AST/lexeme scan + exact oracle | 是 | PASS |
| 对称主元顺序稳定 | comparator 3 repeats + source review | 是 | PASS |
| negative pivot 与 zero-pivot residual coupling 均 fail-closed | 两个独立 oracle + 854 矩阵穷举 | 是 | PASS |
| formula 使用 exact `Fraction`，未舍入结果位于 `[1,n]` | analytic oracle + invariant tests | 是 | PASS |
| half-even 只调用一次，舍入后再次检查 `[1,n]` | monkeypatch call counter + failure injection | 是 | PASS |
| method/hash/input lineage 进入 stable identity | S01 trust-binding 回归 + repeat hash | 是 | PASS |
| 无 float/tolerance/clamp/random pivot | imports/lexeme scan | 是 | PASS；命中=0 |
| exact integer growth 不被包装为 production SLA | source/CP6 risk/packet 语义审查 | 是 | PASS；无 cap、无 SLA、无 production claim |

## 6. 分层验证与执行证据

| 层 | 命令/检查 | 结果 |
|---|---|---|
| Unit + regression | `uv run --python 3.11 pytest -q ...evidence_contracts.py ...estimator.py` | PASS，`87 passed in 0.18s` |
| Exact adversarial oracle | n=3 五值网格 125 例、n=4 三值网格 729 例；与所有主子式非负的 exact PSD oracle 比对 | PASS，854/854；mismatch=0 |
| Syntax | `uv run --python 3.11 python -m py_compile`（3 个模块） | PASS，3/3 |
| Diff hygiene | `git diff --check --`（S01/S02 source/test） | PASS，whitespace error=0 |
| Numeric/static boundary | `rg` 扫描 float/tolerance/clamp/random/numpy/scipy 及 I/O/credential/runtime 入口 | PASS，命中=0 |
| 人工语义审查 | 公式、failure precedence、proof branch、range、claim ceiling、性能边界 | PASS |

## 7. Findings

无 BLOCKER/HIGH/MEDIUM/LOW 实现 finding；不需要回修。

## 8. 剩余风险与知情边界

| 风险 | 状态 | Owner / 后续动作 |
|---|---|---|
| `R-CR173-EXACT-INTEGER-GROWTH` | CP5 已接受；本轮验证无新增缺陷 | 当前 fixture 最大 `n=4`，只声明 `O(n³)` time / `O(n²)` space，不声明 production SLA。CR-172 PATH-C/A、真实规模输入或性能目标出现前必须重新设计/压测。 |
| `R-CR173-SECOND-ORDER-MODEL-BIAS` | 已接受、claim 受限 | 输出仅是相关结构的二阶有效维度，不是 Li-Ji effective independent tests，不用于 FWER/DSR/admission calibration。 |
| public C1 projection | 范围外且保持 unavailable | 由 future projection CR 处理；S02 完成不使 C1 computable。 |

这些是已批准边界，不是本轮新增 waiver；本轮 waiver 数为 `0`。

## 9. 阶段决策

`PASS`。S02 满足 CP7 出口，可标记 `verified` 并解锁 S03。最高声明仍是 repository-local `offline_method_ready` 的组成部分；本结论不授权 public C1、真实数据、runtime、Stage 3、admission、发布或远程写入。
