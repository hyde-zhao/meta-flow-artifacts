---
cr_id: "CR-033"
story_id: "STORY-EX-14"
title: "verify_loss 消费侧提取 tx/rx/loss_ratio"
wave: 4
priority: "P3"
lld_policy: "technical-note"
feature_design_refs:
  - "docs/features/case-execution/DESIGN.md"
depends_on: ["ST-EX-12"]
dependency_type: "hard"
status: "lld-ready"
design_evidence_type: "technical-note"
lld_policy_required_level: "technical-note"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-28T16:35:00+08:00"
author: "meta-dev"
---

# ST-EX-14 verify_loss 消费侧提取 tx/rx/loss_ratio

## 设计证据类型

technical-note（extract_payload 按 op_id 提取字段，ST-EX-12 已建立统一函数框架，本 Story 新增 `tg_verify_traffic_loss` 分支，逻辑简单，Story 内技术说明足够）。

## 设计依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §16.1 改进 #9（verify_loss 消费侧提取 tx/rx/loss_ratio） | 本 Story 来源 |
| HLD | §18 下沉内容（extract_payload(op_id, envelope) 按 op_id 字段提取表 -> FE-EX-02 DESIGN.md） | extract_payload 归属 |
| HLD | §11 技术选型（extract_payload 位置放在 case_runner.py，AGA-04=A） | 函数位置 |
| 需求 | R-F-014（envelope 统一解析）/ R-F-020（verify_loss 消费侧） | 字段提取需求 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 4 ST-EX-14 | file_ownership + depends_on |
| 真相源 | skills/trex-traffic/SKILL.md §tg_verify_traffic_loss + references/six-atom-walkthrough.md §4.3 | envelope.data 字段定义 |
| op_mapper | skills/policy-route-execution/scripts/op_mapper.py envelope 结构（op_id/step_name/status/data/error_type） | envelope 顶层结构 |

## 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | `extract_payload.tg_verify_traffic_loss` | 新增分支 | 在 ST-EX-12 的 extract_payload 函数中新增 tg_verify_traffic_loss 分支 |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `skills/case-execution/scripts/case_runner.py#extract_payload.tg_verify_traffic_loss`

**不触碰文件**：
- `skills/case-execution/scripts/case_runner.py#extract_payload`（函数框架与分派逻辑由 ST-EX-12 拥有；本 Story 只新增 tg_verify_traffic_loss 分支）
- `skills/policy-route-execution/scripts/op_mapper.py`（envelope 由 op_mapper execute_op 产生，本 Story 只消费不改产生侧）
- `skills/trex-traffic/SKILL.md`（envelope.data 字段定义的真相源，不改）

**与 ST-EX-12 边界澄清**：ST-EX-12 建立 `extract_payload(op_id, envelope) -> dict` 统一函数框架（含 op_id 分派机制 + 公共字段提取 + 错误处理），ST-EX-14 在其中新增 `tg_verify_traffic_loss` 一个分支。两者同属 `extract_payload` 函数，文件归属按 DEVELOPMENT-PLAN.yaml 划分：ST-EX-12 拥有函数框架，ST-EX-14 拥有 tg_verify_traffic_loss 分支。实施时 ST-EX-14 在 ST-EX-12 框架内追加分支，不修改框架签名。

## 接口设计要点

### envelope 结构（tg_verify_traffic_loss）

ptm-atomic 返回 envelope 顶层结构（op_mapper execute_op 产出）：

| 字段 | 类型 | 说明 |
|---|---|---|
| op_id | str | "tg_verify_traffic_loss" |
| step_name | str | 步骤名称 |
| status | str | "success" / "failed" |
| data | dict | 业务数据（见下表） |
| error_type | str | "NONE" / "STATE_MISMATCH" / "RESOURCE_NOT_FOUND" / ... |

`data` 字段（真相源：trex-traffic/SKILL.md + six-atom-walkthrough.md §4.3）：

| 字段 | 类型 | 说明 | ST-EX-14 提取 |
|---|---|---|---|
| passed | bool | loss_ratio <= max_loss 且 tx_packets > 0 | 是（辅助） |
| tx_packets | int | TX 端口发包数 | **是（核心）** |
| rx_packets | int | RX 端口收包数 | **是（核心）** |
| loss | int | 丢失包数 = tx - rx | 是（辅助） |
| loss_ratio | float | 丢包率 = loss / tx | **是（核心）** |
| max_loss | int/float | 阈值（比例，0=不允许丢包） | 是（辅助） |

### extract_payload.tg_verify_traffic_loss 提取逻辑

```python
# extract_payload(op_id, envelope) 统一函数中 tg_verify_traffic_loss 分支
# op_id == "tg_verify_traffic_loss":
data = envelope.get("data") if isinstance(envelope.get("data"), dict) else {}
return {
    "passed": data.get("passed"),              # bool, 流量校验结论
    "tx_packets": data.get("tx_packets"),      # int, 发包数（任务要求的 tx）
    "rx_packets": data.get("rx_packets"),      # int, 收包数（任务要求的 rx）
    "loss": data.get("loss"),                  # int, 丢包数
    "loss_ratio": data.get("loss_ratio"),      # float, 丢包率（任务要求的 loss_ratio）
    "max_loss": data.get("max_loss"),          # int/float, 阈值
}
```

**字段名映射说明**：任务要求"提取 tx/rx/loss_ratio"对应 envelope.data 的 `tx_packets`/`rx_packets`/`loss_ratio`（ptm-atomic 真相源字段名）。`tx`/`rx` 是任务简称，实现时用 `tx_packets`/`rx_packets` 精确字段名，不重命名。

### 消费方

| 消费方 | 用途 | 关联 Story |
|---|---|---|
| 四态分级（classify_result） | passed=true -> PASS；passed=false 且 known_issue -> KNOWN_FAIL；passed=false -> FAIL | ST-EX-10 |
| 报告生成（generate_report） | report.md 含 tx/rx/loss_ratio/loss_ratio vs max_loss | ST-EX-11 |
| 失败诊断（diagnose_failure） | STATE_MISMATCH 时附 tx/rx/loss_ratio 上下文 | ST-EX-11 |

### 失败行为

| envelope 状态 | 提取行为 | 返回 |
|---|---|---|
| status=success, data 完整 | 正常提取 | {passed, tx_packets, rx_packets, loss, loss_ratio, max_loss} |
| status=success, data 缺字段 | 缺失字段填 None | {passed:None, tx_packets:None, ...} |
| status=failed, error_type=STATE_MISMATCH | 仍提取 data（含 passed:false） | {passed:false, tx_packets, rx_packets, loss_ratio, ...} |
| status=failed, error_type=RESOURCE_NOT_FOUND | data 可能为空 | {passed:None, tx_packets:None, ...} |
| data 非 dict | 返回全 None 字典 | {passed:None, ...} |

**关键**：envelope status=failed 时仍尝试提取 data（STATE_MISMATCH 时 data 含 passed:false + 统计字段），供四态分级与诊断消费。不因 status=failed 抛异常。

## 实施步骤

1. **确认 ST-EX-12 框架就绪**：`extract_payload(op_id, envelope)` 函数已由 ST-EX-12 建立，含 op_id 分派机制（如 if/elif 或 dict dispatch）
2. **新增 tg_verify_traffic_loss 分支**：在 extract_payload 的 op_id 分派中追加 `"tg_verify_traffic_loss"` 分支，按 §接口设计要点 提取 6 字段
3. **字段名对齐真相源**：用 `tx_packets`/`rx_packets`/`loss_ratio`/`loss`/`max_loss`/`passed`，与 trex-traffic/SKILL.md envelope.data 一致，不重命名
4. **失败行为处理**：data 缺字段填 None，data 非 dict 返回全 None 字典，不抛异常
5. **消费方对接验证**：确认 classify_result（ST-EX-10）能读 passed；generate_report（ST-EX-11）能读 tx_packets/rx_packets/loss_ratio
6. **dry-run 校验**：用 tg_verify_traffic_loss 的 mock envelope 验证提取结果

## 回滚策略

- 本 Story 只在 extract_payload 中新增一个 op_id 分支，不修改函数签名或框架
- 回滚方式：删除 tg_verify_traffic_loss 分支，extract_payload 对该 op_id 返回空字典或触发 UNKNOWN_OP_ID
- 回退影响：四态分级与报告生成无法消费 verify_loss 字段，退化为 PASS/FAIL 二态（可接受，不阻断执行）

## 测试要点

| 测试项 | 验证方法 | 预期结果 |
|---|---|---|
| success envelope 提取 | mock envelope {status:success, data:{passed:true, tx_packets:500, rx_packets:500, loss:0, loss_ratio:0.0, max_loss:0}} | 返回 6 字段全有值 |
| STATE_MISMATCH 提取 | mock envelope {status:failed, error_type:STATE_MISMATCH, data:{passed:false, tx_packets:500, rx_packets:480, loss:20, loss_ratio:0.04, max_loss:0}} | 返回 passed:false + 统计字段 |
| data 缺字段 | mock envelope {status:success, data:{passed:true}} | 缺失字段 None，passed=true |
| data 非 dict | mock envelope {status:success, data:null} | 返回全 None 字典，不抛异常 |
| RESOURCE_NOT_FOUND | mock envelope {status:failed, error_type:RESOURCE_NOT_FOUND, data:{}} | 返回全 None 字典 |
| 字段名对齐 | 提取结果 key 与 trex-traffic/SKILL.md envelope.data 一致 | tx_packets/rx_packets/loss_ratio（非 tx/rx） |
| 消费方对接 | classify_result 读 passed；generate_report 读 tx_packets/rx_packets/loss_ratio | 四态分级与报告正确 |


## 权限与风险

- **权限**：NO_CREDENTIAL_READ（不读凭据）/ NO_PRODUCTION_WRITE（dry-run 默认门，不写生产）/ NO_EXTERNAL_PUBLISH（不发布）
- **失败行为**：envelope.data 字段缺失 -> tx/rx/loss_ratio 填 None 不抛异常，passed 按 envelope.status 判定
- **风险**：loss_ratio 字段名变更或 envelope 结构调整导致提取失败，需同步 _PAYLOAD_FIELDS
## 开放项

无。envelope.data 字段定义已在 trex-traffic/SKILL.md + six-atom-walkthrough.md §4.3 定稿，extract_payload 框架由 ST-EX-12 承载，本 Story 只追加一个分支，无阻塞问题。

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 | 一致性 |
|---|---|---|
| HLD §16.1 改进 #9（verify_loss 提取 tx/rx/loss_ratio） | §接口设计要点 提取 tx_packets/rx_packets/loss_ratio | 一致（tx/rx 简称 -> tx_packets/rx_packets 精确字段名） |
| HLD §18（extract_payload 按 op_id 字段提取表 -> FE-EX-02） | §文件影响范围（case_runner.py extract_payload） | 一致 |
| HLD §11（extract_payload 位置放在 case_runner.py，AGA-04=A） | §文件影响范围 | 一致 |
| R-F-014（envelope 统一解析） | extract_payload tg_verify_traffic_loss 分支 | 一致 |
| R-F-020（verify_loss 消费侧） | 提取 tx/rx/loss_ratio 供四态分级与报告消费 | 一致 |
| op_mapper envelope 结构（op_id/status/data/error_type） | §接口设计要点 envelope 结构 | 一致 |
| trex-traffic/SKILL.md envelope.data 真相源 | §接口设计要点 data 字段表 | 一致（字段名不重命名） |
