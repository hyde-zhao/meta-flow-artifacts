---
discussion_id: CP3-CR173-HLD-DISCUSSION
change_id: CR-173
stage: CP3
owner: meta-se-critical
status: completed-awaiting-host-gate
created_at: "2026-07-16T13:31:00+08:00"
context_ref: process/context/CP3-CR173.context.json
---

# CP3 CR-173 HLD Discussion Log

## 修订记录

| 版本 | 时间 | 修订人 | 变更 |
|---|---|---|---|
| 1.0 | 2026-07-16 13:31 +08:00 | meta-se-critical | 记录 Architecture Gray Areas、advisor table、方法与 public-contract split 决策输入。 |

## 讨论边界

本轮按 Host handoff 执行 relay 模式，meta-se 不直接发起正式人工门禁。输入来自 CP2 已批准的 8 个 DQ、CR173 产品章节和只读源码 call-path inventory；未伪造独立 reviewer agent 意见。禁止 Story/LLD/实现、真实数据/runtime 和 public C1 写入。

## Architecture Gray Areas

| ID | 灰区 | 影响 | 处理 |
|---|---|---|---|
| AGA-01 | 二阶 effective dimension vs alpha-specific FWER equivalent | 方法/输入/claim | 推荐 participation ratio；FWER 转 Spike |
| AGA-02 | estimator 是否估计 correlation matrix | owner/authz/bias | 否；只消费上游 sealed matrix |
| AGA-03 | 七字段中如何保存 reason | schema/audit | status 内嵌 state+reason_code |
| AGA-04 | public C1 是否可 additive projection | compatibility/rollback | 8/8+12/12 inventory 证明非兼容，拆分 |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| participation ratio estimator-only | 可识别、确定、严格边界、权限最小 | 非 FWER/DSR calibration；public consumer 延后 | method/schema/fixture/docs | 推荐 | sealed PSD matrix 可提供；alpha/tail 目标触发 Spike |
| alpha-specific Gaussian-copula equivalent | 更贴近 FWER | tail/copula/alpha 与数值积分未识别 | method/numerics/validation | 条件备选 | 新 use case 与统计输入全部冻结后 |
| Spike-only | 最保守 | 无 offline count 价值 | schedule/scope | 回退 | 推荐方法不被 CP3 接受 |
| estimator + direct C1 modification | consumer value 一轮交付 | int/4-field→decimal/7-field；跨 4 owner 域 | lineage/statistics/Gate1/admission | 禁止本 CR | future versioned migration CR |

## 方案形成输入

| 视角 | 输入 | 影响 | 结果 |
|---|---|---|---|
| product | UC/REQ/DQ approved | non-alias、strategy-agnostic、claim ceiling | adopted |
| architecture | formula proof + source inventory | estimator-only、public split | adopted |
| quality | 8 scenarios、6×3、failure semantics | canonical numeric、append-only recovery | adopted |
| docs | seven fields vs legacy aliases | standalone≠public C1 | adopted |

## 关键结论

- `DO-CR173-CP3-001=PASS`：estimand、input mapping、valid domain、identifiability、assumptions、model/perturbation bias boundary 已冻结。
- `DO-CR173-CP3-002=PASS_BY_SPLIT`：production `8/8`、regression/authorization `12/12`；classification=`cross-owner+cross-domain+non-compatible`。
- 推荐 method ID=`spectral_participation_ratio`；不含 FWER/DSR/admission 暗示。
- matrix 由上游显式提供，CR173 不估计；empirical sampling/stability 留 future activation。
- 数值唯一性：≤12-decimal exact rational、无 tolerance deterministic LDLᵀ PSD、12 位 half-even、双范围检查、无 Decimal→float。
- HLD split：当前仅 estimator；public projection 是 future CR candidate，不创建未授权 companion HLD。

## 场景模拟

| ID | 场景 | 结果 |
|---|---|---|
| SIM-01 | n=4 independent | 4，PASS |
| SIM-02 | n=4、equicorrelation 0.5 | 2.285714285714，PASS |
| SIM-03 | NaN 或 legacy public projection | unavailable / split，raw fallback=0，PASS |

## 待 Host 收集的 CP3 决策

| DQ | 推荐 | 备选 | 状态 |
|---|---|---|---|
| DQ-CR173-CP3-001 | 批准 participation-ratio estimator 与限定 claim | methodology Spike、全量 unavailable | pending-host-human-gate |
| DQ-CR173-CP3-002 | 批准 estimator-only split，projection 后续 CR | 暂停 CR173 | pending-host-human-gate |

阻断问题=`0`；非阻断 open=`2`：future empirical matrix error/stability、future public schema migration。正式人工门禁、STATE/GATE ledger 更新由 Host Orchestrator 负责。
