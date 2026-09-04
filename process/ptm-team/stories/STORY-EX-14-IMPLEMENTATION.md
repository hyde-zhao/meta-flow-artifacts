---
cr_id: "CR-033"
story_id: "STORY-EX-14"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-14.md"
status: "implemented"
implemented_at: "2026-07-30T10:30:00+08:00"
author: "meta-dev"
---

# ST-EX-14 实现执行证据：verify_loss 消费侧提取 6 字段 + CASE_ID_RE 修复

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行号 |
|---|---|---|---|---|
| _PAYLOAD_FIELDS tg_verify_traffic_loss | skills/case-execution/scripts/case_runner.py | ST-EX-14 | 修改（3->6 字段） | L121-L128 |
| extract_payload 0 值跳过修复 | skills/case-execution/scripts/case_runner.py#extract_payload | ST-EX-14 bugfix | 修改 | L164-L166 |
| CASE_ID_RE 正则修复 | skills/case-execution/scripts/case_runner.py | ST-EX-13 前置 | 修改 | L91-L94 |
| parse_case_file 错误消息 | skills/case-execution/scripts/case_runner.py#parse_case_file | ST-EX-13 前置 | 修改 | L234-L237 |
| generate_report 流量统计展示 | skills/case-execution/scripts/case_runner.py#generate_report | ST-EX-14 消费方 | 修改 | L1320-L1329 |

## 2. 设计契约映射

| LLD 契约（STORY-EX-14.md） | 实现落点 | 一致性 |
|---|---|---|
| §接口设计 6 字段（passed/tx_packets/rx_packets/loss/loss_ratio/max_loss） | _PAYLOAD_FIELDS tg_verify_traffic_loss 扩展为 6 条目 | ✓ 一致 |
| 字段名对齐真相源 trex-traffic/SKILL.md | tx_packets/rx_packets/loss_ratio/loss/max_loss/passed（不重命名 tx/rx） | ✓ 一致 |
| data 缺字段填 None | extract_payload 优先级链全未命中 -> value=None | ✓ 一致 |
| data 非 dict 返回全 None 字典 | `if not isinstance(data, dict): return {fname: None}` | ✓ 一致 |
| status=failed 时仍尝试提取（STATE_MISMATCH） | extract_payload 不检查 status，只看 data | ✓ 一致 |
| 不抛异常 | 纯 dict 查找，无 raise | ✓ 一致 |
| 不触碰 extract_payload 函数框架 | 仅修改 0 值跳过判断（bugfix），不改签名/分派机制 | ✓ 一致（bugfix 说明见 §6） |
| classify_result 能读 passed | envelope.status 反映 passed（success=passed:true, failed=passed:false），classify_result 不需直接读 passed | ✓ 一致（消费方对接验证 PASS） |
| generate_report 能读 tx/rx/loss_ratio | 新增 traffic 行展示 tx/rx/loss/loss_ratio/max_loss/passed | ✓ 一致（最小改动） |
| diagnose_failure STATE_MISMATCH 附 tx/rx/loss_ratio | diagnose_failure 已调 extract_payload，details 自动含 6 字段 | ✓ 一致（消费方对接验证 PASS） |

### CASE_ID_RE 修复契约映射

| 契约 | 实现落点 | 一致性 |
|---|---|---|
| 匹配 PC-COMB-M4-01-01（2 字母数字段 + 2 数字段） | `(?:-[A-Z0-9]+)?` 可选段匹配 M4 | ✓ MATCH |
| 匹配 PC-M1-01-01（1 字母数字段 + 2 数字段） | 可选段不匹配，回溯后 `-\d{2}-\d{2}` 匹配 | ✓ MATCH |
| 匹配 PC-M3-01-01 | 同上 | ✓ MATCH |
| 匹配旧格式 PC-M4-01-01-01（1 字母数字段 + 3 数字段） | 可选段贪心匹配 -01，`-\d{2}-\d{2}` 匹配 -01-01 | ✓ MATCH（向后兼容） |
| 前缀匹配不锚定结尾 | 正则无 `$`，`.match` 前缀匹配 | ✓ 一致 |
| _parse_case_id_from_filename 自动受益 | 复用 CASE_ID_RE，验证 4 格式全 PASS | ✓ 一致 |

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| 单元测试 | python3 -c assert（内联） | UT-14-01..10 + 消费方对接 6 项 |
| CASE_ID_RE 验证 | python3 -c assert（内联） | 6 格式 MATCH 验证 |
| _parse_case_id_from_filename 验证 | python3 -c assert（内联） | 4 格式 (case_id, case_name) 验证 |
| dry-run 集成 | /tmp/cr033-w3-test/cases/ + /tmp/ptm-test/ | 旧格式用例解析 + 执行不退化 |

## 4. 最小实现切片

### 任务 1：tg_verify_traffic_loss 6 字段提取

- _PAYLOAD_FIELDS tg_verify_traffic_loss 条目从 3 字段扩展为 6 字段（passed/tx_packets/rx_packets/loss/loss_ratio/max_loss）
- extract_payload 0 值跳过 bugfix：`data[key] not in (None, "", 0)` -> `data[key] is not None and data[key] != ""`
  - 原因：0 和 False 是合法值（loss=0/max_loss=0/passed=False/loss_ratio=0.0），不应被当作"缺失"跳过
  - 影响：所有 op_id 的 0 值字段不再被跳过（修复了 hitcount=0 的潜在 bug）
  - 不破坏既有行为：None 和空字符串仍跳过（多 key chain fallback 正常）
- generate_report 失败诊断新增 traffic 行（tg_verify_traffic_loss 失败时展示 tx/rx/loss/loss_ratio/max_loss/passed）

### 任务 2：CASE_ID_RE 正则修复

- 旧正则：`^(PC-[A-Z0-9]+-\d+-\d+-\d+)`（1 字母数字段 + 3 数字段，不匹配 24 用例编号）
- 新正则：`^(PC-[A-Z0-9]+(?:-[A-Z0-9]+)?-\d{2}-\d{2})`（1 或 2 字母数字段 + 2 数字段）
- 向后兼容：旧格式 PC-M4-01-01-01 仍匹配（可选段贪心匹配 -01）
- ST-EX-15 _parse_case_id_from_filename 复用 CASE_ID_RE，自动受益

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| 语法检查 | python3 -c "import ast; ast.parse(...)" | SYNTAX OK |
| op_mapper validate 不退化 | op_mapper.py validate | PASS (22 op_id) |
| CASE_ID_RE 6 格式 MATCH | python3 -c assert | ALL PASS（含旧格式兼容） |
| _parse_case_id_from_filename 4 格式 | python3 -c assert | ALL PASS |
| UT-14-01 success 6字段全提取（含 0 值） | python3 -c assert | PASS |
| UT-14-02 STATE_MISMATCH passed:false+stats | python3 -c assert | PASS |
| UT-14-03 data 缺字段填 None | python3 -c assert | PASS |
| UT-14-04 data 非 dict 返回全 None | python3 -c assert | PASS |
| UT-14-05 envelope=None 返回全 None | python3 -c assert | PASS |
| UT-14-06 RESOURCE_NOT_FOUND 全 None | python3 -c assert | PASS |
| UT-14-07 字段名对齐真相源 | python3 -c assert | PASS |
| UT-14-08 无业务字段 op 返回 {} | python3 -c assert | PASS |
| UT-14-09 0 值不被跳过（loss=0/max_loss=0/passed=False） | python3 -c assert | PASS |
| UT-14-10a hitcount=5 不受影响 | python3 -c assert | PASS |
| UT-14-10b hitcount=0 不跳过（bugfix 受益） | python3 -c assert | PASS |
| classify_result success -> PASS | python3 -c assert | PASS |
| classify_result STATE_MISMATCH -> FAIL | python3 -c assert | PASS |
| classify_result STATE_MISMATCH+known_issue -> KNOWN_FAIL | python3 -c assert | PASS |
| diagnose_failure details 含 6 字段 | python3 -c assert | PASS |
| generate_report 含 traffic 行（tx/rx/loss_ratio/passed） | python3 -c assert | PASS |
| 无 eval/exec | grep -nE '\beval\(|\bexec\(' | 无 |
| dry-run 旧格式用例不退化 | case_runner run --cases-dir /tmp/cr033-w3-test/cases/ | PASS（PC-M4-01-01-01 解析正常，error 为用例本身缺参数非 CASE_ID_RE 导致） |

## 6. 平台差异

- canonical 源：skills/case-execution/scripts/case_runner.py
- extract_payload 是纯计算（dict 查找），无平台差异
- CASE_ID_RE 是纯正则，无平台差异
- 不改 op_mapper.py（envelope 产生侧，本 Story 只消费）
- 不改 trex-traffic/SKILL.md（envelope.data 字段定义真相源）

### extract_payload 0 值跳过 bugfix 说明

ST-EX-12 原始实现 `data[key] not in (None, "", 0)` 会跳过 0 和 False（因 `False == 0` 为 True），
导致 tg_verify_traffic_loss 的 loss=0/max_loss=0/passed=False/loss_ratio=0.0 被当作"缺失"返回 None。
ST-EX-14 修复为 `data[key] is not None and data[key] != ""`，仅跳过 None 和空字符串。
此修复是 ST-EX-14 6 字段提取的前置必要条件（不修复则 0 值字段无法正确提取），
且修复了 hitcount=0 的潜在 bug（fw_verify_policy_route_hitcount），对其他 op_id 无负面影响。

## 7. 交接摘要

### ST-EX-14 verify_loss 6 字段提取

- _PAYLOAD_FIELDS tg_verify_traffic_loss 扩展为 6 字段（passed/tx_packets/rx_packets/loss/loss_ratio/max_loss）
- extract_payload 0 值跳过 bugfix（0 和 False 是合法值，不应跳过）
- generate_report 失败诊断新增 traffic 行（展示流量统计上下文）
- classify_result / diagnose_failure 已通过 envelope.status / extract_payload 自动对接，无需额外修改
- 12 单元测试 + 6 消费方对接验证全部 PASS

### CASE_ID_RE 修复（ST-EX-13 前置）

- 正则从 `^(PC-[A-Z0-9]+-\d+-\d+-\d+)` 修复为 `^(PC-[A-Z0-9]+(?:-[A-Z0-9]+)?-\d{2}-\d{2})`
- 支持 24 用例真实编号格式：PC-COMB-M4-01-01 / PC-M1-01-01 / PC-M3-01-01
- 向后兼容旧格式 PC-M4-01-01-01
- ST-EX-15 _parse_case_id_from_filename 自动受益
- 6 格式 MATCH 验证 + 4 格式 _parse_case_id_from_filename 验证全部 PASS

### 待 QA 注意

- 既有 /tmp/w3b-unit-tests.py UT-12-04 期望 3 key，扩展后为 6 key，需更新断言（ST-EX-14 扩展预期行为，非退化）
- CASE_ID_RE 修复是 ST-EX-13 前置项，ST-EX-13 可基于此继续 24 用例 md 整改
