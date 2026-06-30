---
checkpoint: CP5
story_id: CR139-S30
story_slug: runid-naming-convention
lld_policy: technical-note
batch_id: CR139-W2-DATA-CONTRACTS
auto_precheck: true
status: ready-for-review
created_at: "2026-06-29T07:10:07+08:00"
created_by: "host-orchestrator-inline-fallback"
---

# CP5-CR139-S30 LLD Implementability Precheck — N2+N3 run_id 命名规约统一

## Entry Criteria

- [x] Story 卡片存在并包含 `## 技术说明`
- [x] lld_policy=technical-note，符合 MATRIX v1.13 REQ-202/203
- [x] 依赖 S08 已 verified
- [x] 物理分区迁移未授权

## Checklist

| # | 检查项 | 结果 | 证据 |
|---|---|---|---|
| 1 | run_id 格式、unknown 阻断、CR 编号禁止新增明确 | PASS | 技术说明 §接口 / 数据 / 权限变化 |
| 2 | 文件影响范围覆盖 normalization.py / lake_layout.py | PASS | 技术说明 §文件影响范围 |
| 3 | legacy path 只读兼容，不迁移 | PASS | 技术说明 §异常和回退 |
| 4 | fixture/static 测试入口明确 | PASS | 技术说明 §测试入口 |
| 5 | 禁止真实 lake 写入和物理迁移 | PASS | 技术说明 + 不授权声明 |

## Exit Criteria

- [x] S30 technical-note ready-for-review
- [x] 无 blocks_lld=true 未回答项
- [ ] 待正式 CP5 人工确认

## 结论

PASS（自动预检通过，待正式 CP5 批次人工确认；不授权物理分区迁移。）
