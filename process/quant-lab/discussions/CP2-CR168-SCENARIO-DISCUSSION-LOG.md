---
cr_id: CR-168
stage: requirement-clarification
status: ready-for-cp2
created_at: "2026-07-13T17:25:00+08:00"
created_by: host-orchestrator-inline
interaction_mode: user-review-plus-inline-fallback
---

# CP2 CR168 场景讨论日志

## 讨论上下文

用户提交了完整 CR168 启动目标和一轮代码评审结论，并明确要求按评审意见启动；随后明确要求不拉起子 Agent、继续推进。该输入构成标准模式下的用户可见 SGQ 交互，不以后台推断替代。

## 已确认的评审修正

| SGQ | 用户输入摘要 | 复述确认 | 状态 |
|---|---|---|---|
| SGQ-CR168-000 | “按上述 4 处调整后即可启动”；根据提示词启动 CR168。 | 采用 Gate 4 联合门禁、增加 `cost_underestimation_status`、使用 daily/ML multi-strategy fixture、精确跨字段 basis 失败条件；不改变 fixture/static C3 方向。 | confirmed |

## CP2 待决策灰区

| SGQ | 推荐方案 | 备选方案 | 影响 | 状态 |
|---|---|---|---|---|
| SGQ-CR168-001 | fee/tax/spread/slippage/impact approximation 全分解，仅用显式静态参数 | impact 延后，只做 fee/tax/slippage | 决定 schema、reason、fixture 和 HLD 方法模块 | selected-for-CP2 |
| SGQ-CR168-002 | 冻结 C3/C4 最小共享 header，C4-exclusive fields reserved | C3 完全独立，C4 后续迁移 | 决定兼容成本与字段 owner | selected-for-CP2 |
| SGQ-CR168-003 | 1 条 C3-to-Gate-4 projection；C4 unavailable 映射为 absent-no-na-reason；字段级/通用 reason 逃逸由 projection 阻断；不修改 canonical Gate 4 | projection 全部延后 FU-007 | 决定本 CR 的 existing-consumer 集成面 | revised-by-user / selected-for-CP2 |
| SGQ-CR168-004 | daily synthetic + daily/ML compatibility 2 族；event N/A | 只保留 daily fixture | 决定 strategy compatibility 与验证面 | selected-for-CP2 |
| SGQ-CR168-005 | 保持 Stage2 complete / Stage3 not-started 和全部真实/runtime/C4/event/CR155 promotion false/0 | 无可接受扩大选项；扩大必须另行授权 | 决定 release claim ceiling 与安全边界 | selected-for-CP2 |

## CP2 修改意见回写（2026-07-13）

- 用户输入类型：`修改:`，不是 `approve`。
- 已修改：`DQ-CP2-CR168-GATE4` 推荐方案增加 projection-side absent-no-na-reason guard；新增 `SC-CR168-B02` 覆盖字段级与通用 na-reason 逃逸。
- 未修改：其余 4 项 DQ 的推荐方案、9 项需求、15 项 QAC、10 类 C3 输入 fail-closed、2 个 fixture 族和所有不授权边界。
- 前向义务：F2/F3/F4/F7 进入 CP3；F6 只冻结 component/envelope hash domain，不增加跨 strategy type 完整 hash QAC。
- 风险：F5 登记 `R-CR168-VERIFIER-INDEPENDENCE`，非阻断，CP8 必须暴露。
- 当前状态：修订已应用，五项 DQ 仍需统一重新 `approve`；CP2 前继续禁止 HLD/Story/LLD/实现/验证。

## Deferred Ideas

- C4 calculator：`FU-CR161-005`。
- C1-C4 aggregate orchestration / final admission integration / CR155 综合 regression：`FU-CR161-007`。
- 真实 TCA、impact calibration、真实数据连接、runtime：独立高风险 CR/授权。
- event-specific producer：独立 event semantics CR。

## 当前结论

- 场景充分性：可进入 CP1 自动检查。
- 设计就绪：false；CP2 未批准。
- 禁止下一步：HLD/CP3、Story、LLD、实现、验证。
