---
status: draft
version: "1.0"
feature_id: "FEAT-EI-CORE"
---

# Feature Test Plan: Evidence Integrity Core

| Case | 覆盖 | 层级 | 期望 |
|---|---|---|---|
| CT-CORE-01 | chronology/conditional approval/future gate | unit+fixture | 非法顺序 100% reject；合法可 replay |
| CT-CORE-02 | typed ID collision/dangling/cycle | unit | 结构化 finding 含 object/field/ref/route |
| CT-CORE-03 | D2 config exists but D0 absent | contract | CONFIG_VALIDATED；不得 PROFILE_DISCOVERED |
| CT-CORE-04 | selector/request/receipt profile/hash/model/effort mismatch | contract | verified=false；required BLOCKED |
| CT-CORE-05 | thread profile mutation/default→critical | contract | NEW_SPAWN_REQUIRED |
| CT-CORE-06 | CP result final correlation/hash mismatch | integration | wrong final/attempt/hash reject |
| CT-CORE-07 | A-baseline vs Conditional-B | integration | capability incomplete stays A；17+minor fixtures才可 B |
| CT-CORE-08 | `PC-18-D0-FRESHNESS-REPROBE` | contract | expiry/session/epoch/hash/schema/reload每项均使旧 probe stale并要求 re-probe |
| CT-CORE-09 | `PC-19-FOLLOWUP-NO-REUSE-RECEIPT` | negative | 原 spawn verified 但 followup receipt缺失时 followup verified/model_attested 均 false |

风险驱动要求：非法 fixture 拒绝率=100%；合法 fixture接受率=100%；identity mutation接受数=0；无 receipt verified数=0。测试命令由 Story LLD 落到 `uv run pytest ...` 与 proposed CLI contract checks；真实平台 conformance unavailable 时明确 NOT_RUN，不以 mock 冒充。

手工审查：核对 CURRENT/REQUIRED 文案、平台 receipt source ownership、A/B切换证据和用户 fallback approval。
