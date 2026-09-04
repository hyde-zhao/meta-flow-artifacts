---
cr_id: "CR-033"
story_id: "STORY-EX-12"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-12.md"
status: "implemented"
implemented_at: "2026-07-30T10:15:00+08:00"
author: "meta-dev"
---

# ST-EX-12 实现执行证据：extract_payload 统一解析

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行号 |
|---|---|---|---|---|
| _PAYLOAD_FIELDS | skills/case-execution/scripts/case_runner.py | ST-EX-12 | 新增常量 | L116-L131 |
| extract_payload | skills/case-execution/scripts/case_runner.py#extract_payload | ST-EX-12 | 新增 | L133-L171 |
| diagnose_failure details 集成 | skills/case-execution/scripts/case_runner.py#diagnose_failure | ST-EX-12 集成点 | 修改 | details 调 extract_payload + _raw_data 兜底 |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §字段提取表（7 op_id） | _PAYLOAD_FIELDS 覆盖 fw_config_policy_route/object/interface/hitcount/tg_verify_traffic_loss/tg_apply_traffic_template/priority | ✓ 一致 |
| op_id 不在表返回 {} | extract_payload `field_specs = _PAYLOAD_FIELDS.get(op_id); if not field_specs: return {}` | ✓ 一致 |
| field 缺失返回 {field: None} | 优先级链全未命中 -> value=None | ✓ 一致 |
| envelope/data 异常返回 {field: None} | envelope 非 dict / data 非 dict -> {fname: None} | ✓ 一致 |
| 优先级链取值（object_id->id） | `for key in key_chain: if key in data and data[key] not in (None,"",0)` | ✓ 一致 |
| 与 _extract_inverse_id 不合并（AGA-04=A） | extract_payload 在 case_runner，_extract_inverse_id 在 op_mapper 保留不动 | ✓ 一致 |
| diagnose_failure details 用 extract_payload | details = dict(payload) if payload else {} + _raw_data 兜底 | ✓ 一致 |
| step-refs 落盘由 ST-EX-04 主干承载 | op_mapper.execute_op 已写 step-refs；extract_payload 供诊断/报告消费 | ✓ 一致（设计说明） |
| dry-run data 空 -> {field: None} 不抛异常 | envelope.data={} -> 优先级链全 None | ✓ 一致 |
| 禁 eval/exec | 纯 dict 查找 | ✓ 一致 |

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| 单元测试 | /tmp/w3b-unit-tests.py | UT-12-01..06 + 3 extra |
| dry-run 集成 | /tmp/w3b-test/cases/ + /tmp/ptm-test/ | fw_config_object 失败诊断 details 含 object_id |

## 4. 最小实现切片

- _PAYLOAD_FIELDS：op_id -> [(字段名, data 键名优先级链)] 映射表（单点维护）
- extract_payload：查表 -> 按优先级链从 envelope.data 取值 -> field 缺失 None -> op 不在表 {}
- diagnose_failure 集成：details = extract_payload(op_id, envelope) + _raw_data 截断兜底

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| 语法检查 | python3 -c "import ast; ..." | SYNTAX OK |
| UT-12-01 policy_route_id | unit_tests.py | PASS |
| UT-12-02 object_id fallback id | unit_tests.py | PASS |
| UT-12-03 hitcount | unit_tests.py | PASS |
| UT-12-04 tx/rx/loss_ratio | unit_tests.py | PASS |
| UT-12-05 无业务字段 op -> {} | unit_tests.py | PASS |
| UT-12-06 envelope 异常 -> {field: None} | unit_tests.py | PASS |
| stream_id fallback template_id | unit_tests.py | PASS |
| field 缺失 -> None | unit_tests.py | PASS |
| envelope=None -> {field: None} | unit_tests.py | PASS |
| dry-run fw_config_object 诊断 | case_runner run --dry-run | details.keys=['object_id','_raw_data'], object_id=null（None 语义） |
| op_mapper validate 不退化 | op_mapper.py validate | PASS (22 op_id) |
| 无 eval/exec | grep | 无 |

## 6. 平台差异

- canonical 源：skills/case-execution/scripts/case_runner.py
- extract_payload 是纯计算（dict 查找），无平台差异
- 不改 op_mapper.py（_extract_inverse_id 保留，职责不同不合并，AGA-04=A）

## 7. 交接摘要

- ST-EX-12 _PAYLOAD_FIELDS + extract_payload 实现 + 9 单元测试通过
- diagnose_failure details 集成 extract_payload（op 特定字段 + _raw_data 兜底）
- step-refs 落盘由 op_mapper.execute_op 主干承载（ST-EX-04），extract_payload 供诊断/报告消费
- O-EX-12-01（_PAYLOAD_FIELDS 同步）OPEN 非阻塞：新增 op 需手动同步表
- O-EX-12-02（tg_apply_traffic_template 键名）OPEN 非阻塞：优先级链 stream_id->template_id 兜底
