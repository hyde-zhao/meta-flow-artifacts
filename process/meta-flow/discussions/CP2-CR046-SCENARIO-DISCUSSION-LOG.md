---
discussion_id: "CP2-CR046-SCENARIO"
cr_id: "CR-046"
stage: "requirement-clarification"
status: completed
owner: "meta-pm"
created_at: "2026-07-11T14:20:00Z"
source: "originating CR request and delegated product-baseline refresh"
---

# CR-046 Scenario Gray Areas Discussion Log

## 场景主体与真实意图

- `engagement_mode=meta-self-dev`
- `scenario_subject_type=implementation-carrier`
- `scenario_subject_id=meta-flow-evidence-integrity`
- 真实意图：让 Meta Flow 不只证明 schema 可通过，还能机器证明时序可信、平台 dispatch 可证、checker 可重放和成本度量诚实。
- 验收 pilot：quant-lab CR-163 仅作为 append-only process-evidence migration/replay 样本；lineage 业务实现不可修改。

## Scenario Gray Areas

| Gray Area | 候选理解 | 推荐 | 已确认输入 | 影响面 | 状态 |
|---|---|---|---|---|---|
| SGA-05 receipt 缺失 | A. 推断 verified；B. 直接失败所有 workflow；C. `unavailable` + 较低证据层级 | C | CR-046 明确要求缺 receipt 不得合成 | provenance / 安全 / 平台兼容 | resolved |
| SGA-06 telemetry 缺失 | A. 估算冒充 measured；B. 不记录；C. measured/proxy/unavailable 分离 | C | CR-046 明确要求缺 telemetry 记 unavailable | 成本 / 审计 / 用户信任 | resolved |
| SGA-07 历史修正 | A. 原地改写；B. append-only correction/supersession；C. 放弃 pilot | B | CR-046 明确要求 CR-163 append-only | 迁移 / 回滚 / 历史不可变 | resolved |
| SGA-08 replay 口径 | A. 只存当前结论；B. 只存执行时结论；C. 双口径 + checker identity/hash | C | CR-046 明确要求 as-executed/current-replay | 兼容性 / 审计 / 发布 | resolved |

## 用户可见确认交互

| Question ID | 问题 | 选项 | 推荐 | 用户响应 | 复述确认 | 状态 |
|---|---|---|---|---|---|---|
| SGQ-CR046-001 | 当平台 receipt 或 token telemetry 缺失、且历史证据需要迁移时，应推断补齐、阻断全部流程，还是诚实降级并 append-only 修正？ | A 推断补齐；B 全部阻断；C unavailable/证据分层 + append-only（推荐） | C | 发起 CR-046 的用户输入明确要求：缺失 receipt/telemetry 记 unavailable、不得合成，CR-163 只做 append-only evidence migration，业务实现 immutable | 本轮产品基线将不可验证事实保持 unavailable，以 correction/supersession 追加事实；不会伪造或修改业务实现 | confirmed |

## Deferred Ideas

- `DEF-EI-001`：跨平台统一加密签名 receipt；等待平台 contract 稳定。
- `DEF-EI-002`：以估算 token 驱动强制计费/配额；等待 telemetry 覆盖与误差模型验证。

## Discussion Close

四个会改变设计、验证和交付边界的灰区均由 originating CR request 明确收敛，无需额外 relay question。该结论只支持 Host Orchestrator 准备 CP2，不等于发起或批准 CP2。
