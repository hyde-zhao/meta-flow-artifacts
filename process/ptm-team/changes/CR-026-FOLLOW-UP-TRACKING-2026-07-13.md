---
change_id: CR-026-FOLLOW-UP-TRACKING-2026-07-13
type: follow-up-tracking
parent_cr: CR-026
created_at: "2026-07-13T18:30:00+08:00"
status: closed
---

# CR-026 Follow-up Tracking - CAP-003 gap 闭环

## gap 闭环登记

| gap ID | 描述 | 状态 | 处理 CR | 闭环证据 |
|--------|------|------|---------|---------|
| CAP-003 | ptm-tde op_id 命名错配追踪（`fw_verify_operation_log` 误代 `fw_capture_operation_log`；op_id 无真相源约束） | resolved-by-CR-026 | CR-026 P0-2 | ptm-tde.md 新增 op_id 选择规则 + 前缀语义（capture/verify 禁令）；run_checkpoint.py GATE-4 新增 op_id 命中校验（`fw_verify_operation_log` 未命中 120 清单 -> BLOCKING） |

## 闭环验证

- `fetch_ptm_atomic_op_ids()` 返回 120 个真实 op_id，`fw_capture_operation_log` 命中、`fw_verify_operation_log` 未命中
- 端到端：构造含 `fw_verify_operation_log` 的 PC -> GATE-4 op_id 命中校验 BLOCKING
- ptm-tde.md:596-601 op_id 选择规则 + 前缀语义表就位

## 后续 CR 候选

| 编号 | 描述 | 优先级 | 状态 | 说明 |
|------|------|--------|------|------|
| F-01 | ptm-te op_mapper.py 简化（args->flag 退化为机械转换） | P1 | candidate | CR-024 follow-up；CR-026 op_id 命中校验为其提供前置 |
| 备选脚本 | `scripts/check_pc_case_steps.py`（type/枚举/required 差集独立校验） | P2 | candidate | 未触发，GATE-4 内联校验已满足；type/枚举留人工确认 |

## 关闭说明

CAP-003 gap 已由 CR-026 P0-2 闭环（op_id 真相源规则 + GATE-4 命中校验）。本 tracking 关闭，无遗留 gap。

## 参考

- CR-026：ptm-tde PC case_steps 契约强化（已 closed）
- CP8 检查结果：`process/checks/CP8-CR-026-DELIVERY-READY.md`
