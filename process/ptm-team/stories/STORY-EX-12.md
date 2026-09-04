---
cr_id: "CR-033"
story_id: "STORY-EX-12"
title: "extract_payload 统一解析（按 op_id 从 envelope 提取字段）"
wave: 3
priority: "P1"
lld_policy: "technical-note"
feature_design_refs:
  - "docs/features/case-execution/DESIGN.md"
depends_on: ["ST-EX-04"]
dependency_type: "hard"
status: "lld-ready"
design_evidence_type: "technical-note"
lld_policy_required_level: "technical-note"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-29T16:00:00+08:00"
author: "meta-dev"
---

# ST-EX-12 extract_payload 统一解析（按 op_id 从 envelope 提取字段）

## 设计证据类型

technical-note（FE-EX-02 required，Feature 级 DESIGN.md 承载 extract_payload 字段提取表；本 Story 内技术说明承载提取逻辑的设计证据）。trigger_reasons：统一解析函数，按 op_id 提取字段；rationale：逻辑简单，Story 内技术说明足够。

## 设计依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §5 推荐方案（case_runner 新增 extract_payload 编排逻辑） | 职责归属 case_runner |
| HLD | §18 下沉（extract_payload(op_id, envelope) 按 op_id 字段提取表 -> FE-EX-02 DESIGN.md） | 字段提取表由 Feature DESIGN.md 承载 |
| HLD | §4 灰区 AGA-04=A（extract_payload 放在 case_runner.py，不放 op_mapper） | 抽象位置决策 |
| 需求 | R-F-014（extract_payload(op_id, envelope) 替代手动 extract_hitcount） | 统一解析函数 |
| 需求 | R-F-020（ST-EX-14 消费侧 verify_loss 提取 tx/rx/loss_ratio） | tg_verify_traffic_loss 字段提取 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 3 ST-EX-12 | file_ownership=case_runner.py#extract_payload，depends_on=ST-EX-04 |
| 现有代码 | op_mapper.py `_extract_inverse_id`（1497 行，从 envelope.data 提取 policy_route_id/interface_id/id） | 提取模式参考；extract_payload 是通用化版本 |

## 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | `extract_payload` | 新增函数 | 按 op_id 从 envelope.data 统一提取业务字段 |
| `skills/case-execution/scripts/case_runner.py` | `_PAYLOAD_FIELDS` | 新增模块常量 | op_id -> 提取字段映射表（单点维护） |
| `docs/features/case-execution/DESIGN.md` | extract_payload 字段提取表 | 补充章节 | 由 ST-EX-04 主导，本 Story 补字段提取表 |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `skills/case-execution/scripts/case_runner.py#extract_payload`

**不触碰文件**：`skills/policy-route-execution/scripts/op_mapper.py`（AGA-04=A，extract_payload 放 case_runner，op_mapper 职责单一不做消费侧解析；op_mapper `_extract_inverse_id` 保留用于 rollback，职责不同不合并）；ST-EX-14 消费侧（`extract_payload.tg_verify_traffic_loss`）由 ST-EX-14 拥有，本 Story 提供通用函数。

## 接口设计要点

### envelope 结构（op_mapper 返回，extract_payload 消费）

op_mapper `_build_envelope`（766 行）返回的 envelope 结构：

```python
{
    "op_id": "fw_config_policy_route",
    "step_name": "配置策略路由",
    "status": "PASS",
    "data": {"policy_route_id": "PR-001", ...},  # ptm-atomic stdout 解析结果
    "error_type": "",
    "diag_snapshot_ref": "",
    "runtime_authorization": {...}  # 仅 --execute 模式
}
```

`data` 字段是 `_parse_atomic_output`（946 行）从 ptm-atomic stdout 解析的 dict，内容随 op_id 变化。extract_payload 按 op_id 从 `envelope.data` 提取关键业务字段。

### _PAYLOAD_FIELDS 字段提取表（单点维护）

| op_id | 提取字段 | data 键名（按优先级） | 用途 |
|---|---|---|---|
| `fw_config_policy_route` | policy_route_id | `policy_route_id` | 后续 step 引用 `${STEP-N.id}`（step-refs） |
| `fw_config_object` | object_id | `object_id` -> `id` | 后续 step 引用（删除/引用对象） |
| `fw_config_interface` | interface_id | `interface_id` -> `id` | 后续 step 引用 |
| `fw_verify_policy_route_hitcount` | hitcount | `hitcount` | 命中计数验证（retry success_condition / 报告） |
| `tg_verify_traffic_loss` | tx_packets, rx_packets, loss_ratio | `tx_packets` / `rx_packets` / `loss_ratio` | 流量损失验证（ST-EX-14 消费侧） |
| `tg_apply_traffic_template` | stream_id | `stream_id` -> `template_id` | 后续 start/stop traffic 引用 |
| `fw_config_policy_route_priority` | policy_route_id | `policy_route_id` -> `id` | 后续引用 |

**无业务字段的 op_id**（如 `fw_login_web_management` / `fw_logout` / `tg_config_interface` / `tg_start_traffic_stream` / `fw_reset_policy_route_hitcount`）不列入 `_PAYLOAD_FIELDS`，extract_payload 返回空 dict `{}`。

**字段优先级**：部分 op_id 的 data 键名不固定（如 fw_config_object 可能返回 `object_id` 或 `id`），按优先级链式查找，取首个非空值。与 op_mapper `_extract_inverse_id`（1515 行 `for key in ("policy_route_id", "interface_id", "id")`）的查找模式一致。

### extract_payload 签名

```python
def extract_payload(op_id: str, envelope: dict) -> dict:
    """按 op_id 从 envelope.data 统一提取业务字段。

    替代散落的手动 extract_hitcount / extract_policy_route_id 逻辑，
    作为 case_runner 消费侧统一入口（AGA-04=A，放在 case_runner.py）。

    Args:
        op_id: 原子操作 ID（如 fw_config_policy_route / tg_verify_traffic_loss）
        envelope: op_mapper.execute_op 返回的 envelope dict

    Returns:
        字段名 -> 值的 dict：
        - op_id 在 _PAYLOAD_FIELDS 中：返回该 op_id 的提取字段（field 不存在返回 None，不抛异常）
        - op_id 不在 _PAYLOAD_FIELDS 中：返回空 dict {}
        - envelope/data 异常：返回空 dict {}（不抛异常，调用方按 None 处理）

    用途：
    - step-refs 落盘：execute_steps 调 extract_payload 取 id 字段写入 step-refs/<step_id>.json
    - retry success_condition 求值：ST-EX-09 _eval_success_condition 从提取结果取字段
    - verify_loss 消费：ST-EX-14 调 extract_payload("tg_verify_traffic_loss", envelope) 取 tx/rx/loss_ratio
    - 报告展示：ST-EX-11 report.md 展示提取的关键字段
    """
```

**与 op_mapper `_extract_inverse_id` 的区别**：
- `_extract_inverse_id`（op_mapper）：提取 rollback 用的 id（单值），服务 handle_rollback；保留不动
- `extract_payload`（case_runner）：提取全部业务字段（多值 dict），服务 step-refs/retry/报告/verify 消费；通用化版本
- 二者职责不同，不合并；op_mapper 保持单一职责（AGA-04=A 决策理由）

### 返回值示例

```python
# fw_config_policy_route 成功
extract_payload("fw_config_policy_route", {"data": {"policy_route_id": "PR-001"}, "status": "PASS"})
# -> {"policy_route_id": "PR-001"}

# fw_config_object（data.object_id 缺失，fallback id）
extract_payload("fw_config_object", {"data": {"id": "OBJ-001"}, "status": "PASS"})
# -> {"object_id": "OBJ-001"}

# tg_verify_traffic_loss（ST-EX-14 消费）
extract_payload("tg_verify_traffic_loss", {"data": {"tx_packets": 100, "rx_packets": 100, "loss_ratio": 0.0}})
# -> {"tx_packets": 100, "rx_packets": 100, "loss_ratio": 0.0}

# fw_login_web_management（无业务字段）
extract_payload("fw_login_web_management", {"data": {"session": "..."}, "status": "PASS"})
# -> {}

# data 缺失/异常
extract_payload("fw_config_policy_route", {"status": "FAIL", "error_type": "EXEC_FAILED"})
# -> {"policy_route_id": None}
```

**None 语义**：op_id 在 `_PAYLOAD_FIELDS` 中但 data 缺该字段时，返回 `{field: None}`（而非省略），调用方可区分"op 应有字段但缺失"与"op 无业务字段"。

### 与 step-refs 的集成

ST-EX-04 execute_steps 执行成功后，调 `extract_payload(op_id, envelope)` 取 id 字段，写入 `runs/<run-id>/step-refs/<step_id>.json`，供后续 step 的 `${STEP-N.id}` 引用（op_mapper `resolve_step_refs` 消费）。step-refs 落盘逻辑由 ST-EX-04 主干承载，本 Story 提供 extract_payload 函数供其调用。

## 实施步骤

1. **定义 `_PAYLOAD_FIELDS` 常量**：op_id -> 字段提取规则（字段名 + 优先级链）；覆盖 R-F-014 列举的 policy_route_id/hitcount/tx_packets/rx_packets + R-F-020 loss_ratio；无业务字段的 op_id 不列入
2. **实现 `extract_payload`**：查 `_PAYLOAD_FIELDS`；op_id 不在表 -> 返回 {}；在表 -> 按优先级链从 envelope.data 取值，field 缺失返回 None；envelope/data 异常返回 {field: None}
3. **接入 step-refs 落盘**：ST-EX-04 execute_steps 成功后调 extract_payload 取 id 字段写入 step-refs（depends_on=ST-EX-04 hard）
4. **接入 retry 求值**：ST-EX-09 _eval_success_condition 可从 extract_payload 结果取字段（或直接从 envelope.data 取，二者一致；推荐 extract_payload 统一入口）
5. **dry-run 行为**：dry-run 模式下 op_mapper 返回 dry_run envelope（data 可能为空），extract_payload 返回 {field: None}，不抛异常；step-refs 不落盘（dry-run 不实际执行）

## 回滚策略

- 本 Story 为 case_runner.py 纯新增函数 + 常量，不修改 op_mapper，不改变 envelope 结构
- 回滚方式：git revert 相关 commit；移除后 ST-EX-04 step-refs 落盘回退到直接从 envelope.data 取 id（与 op_mapper _extract_inverse_id 模式一致），不影响执行
- 无运行时副作用：extract_payload 是纯计算（dict 查找），不触发设备操作

## 权限与风险

- **权限约束**：遵循 NO_CREDENTIAL_READ / NO_PRODUCTION_WRITE / NO_EXTERNAL_PUBLISH；extract_payload 是纯解析不触发设备操作
- **数据安全**：从 envelope.data 提取业务字段（policy_route_id/hitcount/tx_packets 等），不含凭据；session 不入库
- **失败处理**：envelope/data 异常返回 {field: None} 不抛异常；op_id 不在表返回 {}；调用方按 None 处理
- **风险**：纯新增函数 + 常量，无运行时副作用；回滚 git revert

## 测试要点

| 测试项 | 验证方法 | 预期结果 |
|---|---|---|
| policy_route_id 提取 | 单元测试：envelope.data={policy_route_id:"PR-001"} | {policy_route_id:"PR-001"} |
| object_id fallback id | 单元测试：data={id:"OBJ-001"}（无 object_id） | {object_id:"OBJ-001"} |
| hitcount 提取 | 单元测试：data={hitcount:5} | {hitcount:5} |
| tx/rx/loss_ratio 提取 | 单元测试：data={tx_packets:100,rx_packets:100,loss_ratio:0.0} | 三字段 dict（R-F-020） |
| stream_id fallback template_id | 单元测试：data={template_id:"T-01"} | {stream_id:"T-01"} |
| field 缺失 | 单元测试：data={}（op 在表中） | {field: None} |
| 无业务字段 op | 单元测试：op_id=fw_login_web_management | {} |
| envelope 异常 | 单元测试：envelope=None / data=None | {field: None} 或 {} |
| dry-run data 空 | 单元测试：dry_run envelope data={} | {field: None}，不抛异常 |
| _PAYLOAD_FIELDS 覆盖 | 审查常量表 | 覆盖 R-F-014/R-F-020 列举字段 |
| 不引入 eval | grep "eval\|exec" extract_payload | 无 |
| 与 _extract_inverse_id 不冲突 | 审查 op_mapper _extract_inverse_index | 保留不动，职责区分清晰 |

## 开放项

| 问题 ID | 问题 | 状态 | 说明 |
|---|---|---|---|
| O-EX-12-01 | _PAYLOAD_FIELDS 是否随 op_mapper 新增 op 自动同步 | OPEN（非阻塞） | 当前为 case_runner 模块常量，新增 op 需手动同步表。op_mapper 已有 validate_mapping_consistency，后续可扩展校验 _PAYLOAD_FIELDS 覆盖度。CR-033 范围内现有 op 已覆盖 |
| O-EX-12-02 | tg_apply_traffic_template 返回键名未确认 | OPEN（非阻塞） | 现有用例 data 未确认是 stream_id 还是 template_id。优先级链 stream_id -> template_id 兜底；ST-EX-13 整改/ST-EX-14 消费侧验证时确认实际键名，必要时调整优先级 |

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 |
|---|---|
| HLD §5（case_runner 新增 extract_payload） | extract_payload 归属 case_runner |
| HLD §18 下沉（字段提取表 -> FE-EX-02 DESIGN.md） | _PAYLOAD_FIELDS 表补入 DESIGN.md |
| HLD §4 AGA-04=A（extract_payload 放 case_runner） | 放在 case_runner.py，不合并到 op_mapper |
| R-F-014（extract_payload(op_id, envelope) 替代手动 extract） | 统一解析函数 + 字段提取表 |
| R-F-020（ST-EX-14 verify_loss 提取 tx/rx/loss_ratio） | tg_verify_traffic_loss 三字段提取 |
| SM-EX-04（1 个 extract_payload 统一解析函数） | 单一函数 + _PAYLOAD_FIELDS 表 |
