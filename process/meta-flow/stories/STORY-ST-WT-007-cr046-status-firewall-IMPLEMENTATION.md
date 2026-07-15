---
story_id: "ST-WT-007"
cr_id: "CR-047"
stage: "CP6"
status: "implemented_with_risk"
execution_mode: "inline-fallback"
implemented_at: "2026-07-15"
---

# ST-WT-007 Implementation

## 实现对象

- `meta_flow/checks/cr_tracking.py`：实现 CR protected-object manifest 构建与验证；文件按对象身份校验，ledger 只哈希目标 CR event set，允许无关 append。

## Post-approval pre-publication correction

CP8 合法追加 CR-047 Gate event 后发现 selector 曾通过完整序列化正文匹配 `CR-046`，会把“禁止修改 CR-046”的说明文字误当成 CR-046 event。现改为：已有 manifest 使用精确事件 ID；新 manifest 使用结构化 CR 字段或事件 ID。新增 fixture 同时证明无关 append 可通过、受保护事件改写仍失败。
- `process/evidence/CR047-ST-WT-007.protected-originals.json`：CP6 pre-implementation 生成，记录 57 个 CR-046 protected objects 的原始 SHA256。
- `tests/test_cr047_operator_status.py`：证明 prefix-only=false、原件 1 字节变化会产生 blocker finding。
- `process/docs/product/TEST-MATRIX.md`：只更新 current projection，不修改 CR-046 原始 Story/CP/ledger。

## 契约映射

身份键为具体 path/object_type/hash/selector，不使用目录前缀豁免。同目录新增 CR-047 对象不误伤；任何 CR-046 protected original 缺失或变更都必须阻断并路由子 CR。

## 验证与边界

CP6 pre manifest 已生成；实现完成首验要求 findings=0，CP7 将再次验证。current projection 仅允许 `closed / READY_WITH_RISK`，不得声称 receipt、telemetry、独立 QA 或真实 pilot 已具备。

## 设计差异与交接

无 protected-original 变更，未触发子 CR。inline fallback 与 CR-046 继承风险叠加，结论上限保持 CP7 `PASS_WITH_RISK`、CP8 `READY_WITH_RISK`。
