# CP3 CR-037 HLD Discussion Log

## CR-037：项目治理与状态强制整改架构基线

| 字段 | 内容 |
|---|---|
| 日期 | 2026-07-02 |
| 来源 | 已批准 CP2、八轮评审收敛方案、meta-se baseline 设计文档 |
| 关联 CR | `process/changes/CR-037.md` |
| 架构基线 | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` v1.0 |
| 当前状态 | `CR-037` active；CP2 approved；CP3 pending |

## 架构反馈摘要

CP2 已确认 CR-037 的产品 / 场景 / 范围基线。CP3 的目标是确认 CR-037 的蓝图、HLD、ADR 和 Feature Design Matrix 是否能作为后续 per-feature 设计和 CP5 前置输入。当前架构主线为 enforcement-first：先收紧 `STATE.current.json`，再建立 project governance、registry-backed refs、impact normalization、roadmap refresh 和 quant-lab migration readiness。

## Architecture Gray Areas

| 灰区 ID | 问题 | 处理结论 |
|---|---|---|
| AGA-CR037-01 | current state 是黑名单补丁还是 allowlist enforcement？ | 选 allowlist + field budget + audit/enforce，黑名单 strip 被否决。 |
| AGA-CR037-02 | project state 是否进入 `STATE.current.json`？ | 独立 `PROJECT.current.json`；current state 只保存 `project_state_ref`。 |
| AGA-CR037-03 | roadmap refresh 是否跨仓自动写 quant-lab 发布库？ | 否。只自动写过程归档库；发布库输出 stale / FU-RF。 |
| AGA-CR037-04 | capability / feature refs 是否允许自由字符串？ | 否。必须引用标准 registry；缺失 ID blocked 或 FU-RF。 |
| AGA-CR037-05 | ROADMAP-REFRESH 是否复用 CP result？ | 否。使用独立 result schema 和 checker。 |
| AGA-CR037-06 | roadmap follow-up 是否进入 RELEASE-CONTEXT？ | 否。使用 FU-RF / SP-RF / RA-RF，保持项目治理与发布上下文分离。 |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Enforcement-first 分层治理 | 先阻断 current state 污染，后续 project / roadmap / migration 依赖可信入口 | 初期 Story 和 checker 数较多 | state、project-governance、roadmap、registry、migration | 推荐 | 适用于长期治理和真实项目迁移；若只修一次性文件清理才考虑降级 |
| B. Project-governance-first 快速路线 | 较快产生 project objects | current state 仍可能污染，后续返工概率高 | project-governance、state | 不推荐 | 仅当 current state 已被外部机制稳定控制时切换 |
| C. Migration-first 样本驱动路线 | 快速暴露 quant-lab 真实问题 | 容易把样本特例固化成通用机制 | migration、quality、state | 不推荐 | 仅用于 Spike，不适合作为本轮主线 |

## 用户可见确认来源

| Question ID | 来源 | 用户回答 | 架构影响 | 状态 |
|---|---|---|---|---|
| CP2-CR037-DQ-04 | CP2 人工确认 | 确认 P0 使用 allowlist + field budget + audit/enforce | 支撑 ADR-PG-001 | confirmed |
| CP2-CR037-DQ-05 | CP2 人工确认 | 确认 roadmap refresh 只自动写过程归档库机器状态 | 支撑 ADR-PG-003 / ADR-PG-005 | confirmed |
| CP2-CR037-DQ-06 | CP2 人工确认 | 确认 feature_refs / capability_refs 必须引用标准 registry | 支撑 ADR-PG-004 | confirmed |
| CP2-CR037-DQ-07 | CP2 人工确认 | 确认 P2 quant-lab migration 是验证切片，不授权发布库修改 | 支撑 HLD P2 migration boundary | confirmed |

## 下游衔接

- CP3 approve 后，HLD / Blueprint / ADR 可作为 Feature 设计和 Story 拆解输入。
- CP3 approve 不授权实现；required per-feature DESIGN / TEST-PLAN / TASKS 仍需在 CP5 前生成并确认。
- CP3 approve 不关闭、恢复或完成 `CR-036`。
- CP3 approve 不授权 runtime、production write、publish、live、凭据读取或 quant-lab 发布库修改。
