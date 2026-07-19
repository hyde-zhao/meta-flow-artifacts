---
status: complete
version: "1.0"
scope: "CR-172 historical finding closure and active fix status"
created_at: "2026-07-19T01:43:00+08:00"
active_fix_count: 0
waiver_count: 0
---

# Fixes

当前没有需要 meta-dev/meta-se 回修的 active finding；CP8 不创建实现修复。以下历史 NEEDS_REWORK 项完整保留为 CLOSED：

| Fix ID | 来源 Finding | 修复 / 澄清方向 | 路由 | 影响对象 | 复验 / 回归范围 | 状态 |
|---|---|---|---|---|---|---|
| FIX-S01-001 | F-CP7-001 | 单一 decision invariant validator 封闭 constructor/consumer bypass | meta-dev→meta-qa | S01 contract/test | 46/46 | done / CLOSED |
| FIX-S01-002 | F-CP7-002 | canonical URI 封闭 percent-encoding 等价绕过 | meta-dev→meta-qa | S01 contract/test | 46/46 | done / CLOSED |
| FIX-S02-001 | F-CR172-S02-CP7-001 | ordered seal-to-port authorization exact correlation | meta-dev→meta-qa | S02 contract/test | 35/35、81/81 | done / CLOSED |
| FIX-S03-001 | F-CP7-S03-001 | fresh decision strict newer + exact binding | meta-dev→meta-qa | S03 contract/test | 27/27、108/108 | done / CLOSED |
| FIX-S03-002 | F-CP7-S03-002 | concurrent-current CAS 保留另一写者 | meta-dev→meta-qa | S03 contract/test | 27/27、108/108 | done / CLOSED |
| FIX-S04-001 | F-001 | receipt exact-read，失败不推进 selection | meta-dev→meta-qa | S04 contract/test | 19/19、127/127 | done / CLOSED |
| FIX-S04-002 | F-002 | resolver 只接受 storage current exact capability | meta-dev→meta-qa | S04 contract/test | 19/19、127/127 | done / CLOSED |
| FIX-S04-003 | F-003 | sensitive evidence marker 全组合 fail closed | meta-dev→meta-qa | S04 contract/test | 19/19、127/127 | done / CLOSED |
| FIX-S05-001 | F-001 | SC-021→REQ-013 与 SC-006→REQ-006 显式语义绑定 | meta-dev→meta-qa | S05 catalog/test | 27/27、154/154 | done / CLOSED |

## 当前修复结论

- open fixes：`0`。
- blockers：`0`。
- waivers：`0`。
- runtime/deployment gaps 不是本轮实现缺陷，不得通过修改 repository fixture“修复”；它们必须经独立授权、activation-resume CP2 或后续 CR 处理。
- 新发现 `REV-CR172-004` 是 meta-flow checkpoint retry event-ID 生成器的 process-tooling 债务，不回路由 S01～S05 实现；已登记 `FU-CR172-LEDGER-001`。修复前不得原地改写 append-only ledger，也不得宣称 checkpoint ledger 全量检查为 PASS。
