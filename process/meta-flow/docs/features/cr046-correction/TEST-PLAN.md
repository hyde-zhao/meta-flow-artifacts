---
status: draft
version: "1.0"
feature_id: "FEAT-EI-CORRECTION"
---

# Feature Test Plan: Correction Lifecycle and Pilot Adapter

| Case | 触发 | 期望 |
|---|---|---|
| CT-COR-01 | 合法allowlist correction | append并可replay/audit；原hash不变 |
| CT-COR-02 | 非allowlist/缺author reason evidence | reject |
| CT-COR-03 | supersedes dangling/cycle | reject |
| CT-COR-04 | 错误correction的后续修正 | 新superseding event；旧event保留 |
| CT-COR-05 | pilot无授权或protected path diff | BLOCKED；业务写入0 |
| CT-COR-06 | authorized fixture manifest + 23 targets | dry-run/current replay oracle 23/23 |
| CT-COR-07 | legacy codex_agent_name | D3 self-declared-unverifiable；resolved fields unavailable |
| CT-COR-08 | partial append/replay failure | append set隔离/补偿；原历史不改 |

真实CR-163 pilot不在CP4/CP5或本测试计划当前授权内；CT-COR-06使用合成/只读fixture，不能冒充pilot已执行。
