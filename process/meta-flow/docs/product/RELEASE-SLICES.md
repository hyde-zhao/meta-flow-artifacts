---
status: baseline
version: "1.1"
created_at: "2026-07-11"
owner: "meta-pm"
active_change_ref: "CR-046"
source_story_map: "docs/product/STORY-MAP.md"
---

# Meta Flow 产品发布切片

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.0 | 2026-07-11 | meta-pm | 建立独立 release-slice 目录基线；引用既有 SL-PG 切片，并增量记录 CR-046 SL-EI 切片 | 缺失产物初始化；不改变 STORY-MAP 中既有 ID |
| 1.1 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：扩展既有 SL-EI 切片的 compaction/correction/audit/dogfooding/dispatch 验证入口，不改变 Slice 或 Story ID | 原文档增量更新 |

## 既有切片引用

`SL-PG-00..05` 的正式定义继续保留在 `docs/product/STORY-MAP.md`；本文件不复制或重新编号既有切片。后续产品变更应逐步把新切片增量登记在此文件，并保持对 Story Map 的回链。

## CR-046 推荐切片

| Slice ID | 用户价值 | 包含 Story | 前置依赖 | 验证入口 | 风险 / 门控 |
|---|---|---|---|---|---|
| SL-EI-01 | 审计者可以机器证明 gate 时序、真实 dispatch、final CP correlation、compaction 语义与 state/read 一致性 | ST-EI-001..004 | CP2、CP3、全量 CP5 | TC-EI-001..009, TC-EI-014, TC-EI-018 | receipt 缺失必须披露 session/repository 证明上限；compaction 不得丢失关系；不得隐式授权 runtime/credentials |
| SL-EI-02 | 审批者可以区分实测/估算/不可用成本，用机器 audit report 和双口径 replay 复核历史证据 | ST-EI-005..006 | SL-EI-01 | TC-EI-010..012, TC-EI-016..017 | checker provenance 缺失需诚实降级；R1 null-provenance 不得静默改写；legacy YAML 不得干扰 canonical JSON |
| SL-EI-03 | 以通用 post-close correction lifecycle 和真实 CR-163 样本证明 append-only migration/current replay | ST-EI-007 | SL-EI-02；pilot 独立授权 | TC-EI-013, TC-EI-015 | 23/23 PASS；历史不可改写；quant-lab lineage business diff 为 0 |

## 切片边界

- 每个 Story 保持独立 CP6/CP7 verdict；共享 regression 可以批量运行，但不合并 Story 结论。
- CP2 approval 只确认产品/场景/范围，不授权实现、runtime、credentials、publish、commit/push 或 quant-lab business-code 修改。
- `SL-EI-03` 失败时保留 Meta Flow 已验证切片，pilot 回退到设计澄清或实现回修，不覆盖历史证据。
