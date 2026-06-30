---
checkpoint: CP5
story_id: CR139-S31
story_slug: events-schema-repair
lld_policy: technical-note
batch_id: CR139-W2-DATA-CONTRACTS
auto_precheck: true
status: ready-for-review
created_at: "2026-06-29T07:10:07+08:00"
created_by: "host-orchestrator-inline-fallback"
---

# CP5-CR139-S31 LLD Implementability Precheck — C3 events schema 修复

## Entry Criteria

- [x] Story 卡片存在并包含 `## 技术说明`
- [x] lld_policy=technical-note，符合 MATRIX v1.13 REQ-206
- [x] 依赖 S01/S02/S03 已 verified
- [x] 不执行真实 events 重跑

## Checklist

| # | 检查项 | 结果 | 证据 |
|---|---|---|---|
| 1 | events explicit schema 与全 null 阻断明确 | PASS | 技术说明 §接口 / 数据 / 权限变化 |
| 2 | 文件影响范围仅覆盖 normalization.py events 分支 | PASS | 技术说明 §文件影响范围 |
| 3 | 缺字段、全 null、类型不匹配失败路径明确 | PASS | 技术说明 §异常和回退 |
| 4 | fixture/static 测试入口明确 | PASS | 技术说明 §测试入口 |
| 5 | 禁止 runtime 和真实 lake 写入 | PASS | 技术说明 + 不授权声明 |

## Exit Criteria

- [x] S31 technical-note ready-for-review
- [x] 无 blocks_lld=true 未回答项
- [ ] 待正式 CP5 人工确认

## 结论

PASS（自动预检通过，待正式 CP5 批次人工确认；不授权真实 lake 写入。）
