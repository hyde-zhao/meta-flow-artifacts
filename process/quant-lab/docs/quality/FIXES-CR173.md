---
status: complete
version: "1.0"
cr_id: "CR-173"
updated_at: "2026-07-17T12:30:00+08:00"
---

# CR-173 Fixes

| Rework | 修复范围 | 验证 | 状态 |
|---|---|---|---|
| S01 CP6R1 | approved method binding、identifier typed failure、audit recovery 初次收紧 | CP7R1 | 部分关闭，余项转 CP6R2 |
| S01 CP6R2 | bytes/unhashable identifier 稳定映射；constructor/log 双侧复算 content identity | CP7R2 60/60 + 10/10 adjacent probes | CLOSED |
| S03 CP6R1 | AST 授权守卫加入 alias/qualified call/operation 语义绑定，避免 forbidden operation/public call 假阴性 | CP7R1 9/9、124/124、379/379 | CLOSED |

未通过 waiver 绕过任何 finding；未修改已批准的 estimand、schema、claim ceiling 或授权边界。

