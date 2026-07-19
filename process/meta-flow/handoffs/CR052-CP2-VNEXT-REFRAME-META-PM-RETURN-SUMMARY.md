---
handoff_id: "HO-CR052-CP2-VNEXT-REFRAME-META-PM"
return_from: "meta-pm"
return_to: "host-orchestrator"
workflow_id: "meta-flow-self-dev"
change_id: "CR-052"
stage: "requirement-clarification"
status: "complete"
completed_at: "2026-07-19T09:15:23Z"
context_ref: "process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml"
cp1_result_ref: "process/checks/CP1-CR052-USE-CASE-COMPLETENESS-VNEXT-R2.result.json"
cp2_result_ref: "process/checks/CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2.result.json"
next_route: "host-orchestrator-prepares-CP2-human-gate"
---

# CR-052 vNext R2 meta-pm Return Summary

## 返回结论

CR-052 的八份产品基线已按 vNext R2 增量重构完成。旧 ID、旧修订记录和旧结论均未删除；当前对象通过 `reframed`、`superseded`、`deferred` 显式说明与旧 migration-readiness 基线的关系。

- CP1 自动检查：`PASS`。
- CP2 自动预检：`PASS`。
- 产品阻断项：0。
- CP2 人工门：`pending`；5 个决策项和总体基线仍须 host-orchestrator 发起人工确认。
- 当前返回不等于 CP2 `approved`，也不授权 Feature/Story/LLD、源码实现、真实仓创建或迁移、旧仓写入、commit/push、publication、credentials、runtime、production、publish、live 或 trading。

## 当前候选基线

| 对象 | 数量 | 当前范围 |
|---|---:|---|
| Use Case | 6 | `UC-VNEXT-001..006`：双库隔离、四层治理、Work 生命周期、风险路由、scoped 资源控制、快照迁移/只读历史 |
| Requirement | 32 | 功能 21 + 约束 6 + NFR 5；全部为 P0/P1，全部可追溯到 TC-VNEXT |
| Engineering Scenario | 20 | `TC-VNEXT-001..020`；positive 7、negative 4、boundary 4、permission 3、failure-recovery 2 |
| 候选 Story | 6 | `ST-VNEXT-001..006`，仅为 CP2 产品规划候选，不冒充 CP4 Story |
| Release Slice | 4 | `SL-VNEXT-A..D`，依次覆盖双库/四层骨架、Work/CAS、G0/G1/G2 预算、快照试点/只读历史 |
| 成功指标 | 12 | `SM-VNEXT-01..12`，覆盖 route=1、跨项目变化=0、预算、CAS、只读历史和 2×2 试点 |
| Scenario Gray Area | 4 | `SGA-VNEXT-01..04`，均已场景方向确认并路由到正式决策项 |
| 用户可见场景确认 | 1 | `SGQ-VNEXT-001`，只确认 vNext 候选方向，不等于 CP2 批准 |
| 待人工决策 | 5 | `DQ-VNEXT-01..05` |
| Deferred Idea | 5 | `DEF-VNEXT-001..005`，均有重启条件 |

## 推荐默认

1. 每个项目恰好使用一个现有源码/交付仓作为发布库，并新增一个独立过程仓；首版不引入第三仓。
2. 过程仓采用 `main-only` 单写 publisher，并用 `expected-OID CAS` 拒绝陈旧写入。
3. 治理层级固定为 `Project → Roadmap → Phase → Work`；Work 承载一次有界、可验收、可关闭的执行循环。
4. G0 上限为读取 8 个文件、写入 8 个文件、3 个检查组、总 token 32,000；G1 上限为读取 20 个文件、写入 24 个文件、8 个检查组、总 token 96,000；高风险或超过 G1 上限进入 G2。
5. 首版只迁当前快照；旧共享过程仓保持只读索引，不拆分历史、不做旧 CP/CR/Story 无损自动转换。
6. 试点为 2 个项目 × 每项目至少 2 个 Work 周期；每项目至少覆盖 1 个 G0 和 1 个 G1，旧仓只读观察期推荐 30 天。

## 待 host-orchestrator 汇总的人工决策

| 决策 ID | 类型 | 推荐 | 备选 | 主要影响 / 切换条件 |
|---|---|---|---|---|
| DQ-VNEXT-01 | architecture | 复用现有发布仓，每项目只新增独立过程仓 | 另建专用发布仓；或延续共享发布面 | 推荐最符合双库简化目标；仅当现有仓无法承载交付契约且第三仓成本获批准时切换 |
| DQ-VNEXT-02 | architecture | 过程仓 main-only 单写 publisher + expected-OID CAS | per-Work branch；或多写者自动 merge | 推荐减少 working-tree/branch 联动；只有连续两个周期违反已批准吞吐 SLO 且 CAS 重试为主因时重新评估 |
| DQ-VNEXT-03 | scope | 只迁当前快照，旧共享仓永久只读索引 | 按项目拆史；或做旧对象无损转换 | 推荐风险和语义失真最低；仅在强制审计/恢复需求无法由只读索引满足并有新 CR 时切换 |
| DQ-VNEXT-04 | implementation | G0=8/8/3/32k；G1=20/24/8/96k；高风险/超限进 G2 | 更紧预算；或扩大 G1 | 推荐值可直接验收；试点若证明系统性 underfit/overfit，再按观测数据调整 |
| DQ-VNEXT-05 | scope | 2 项目 × 2 周期、每项目覆盖 G0/G1；旧仓观察期 30 天 | 单项目更小试点；或扩大批量试点 | 推荐兼顾样本差异与爆炸半径；试点全部通过后才允许另行批准批量迁移 |

## 历史基线处理

- `PG-*`、`EI-*`、`WT-*`：保留历史并标为 `reframed`，由 VNEXT 的双库、四层和 Work-scoped 模型吸收有效意图。
- `GB-*`、`AW-*`、`MR-*`：保留历史并标为 `superseded`，不再作为当前 CR-052 推荐实现主线。
- 历史拆分、无损转换、多 publisher 自动 merge、批量迁移、未授权远端 publication：标为 `deferred` 或 `out-of-scope`，均未被静默删除。

## 写入清单

### 八份产品基线

1. `process/docs/product/USE-CASES.md`
2. `process/docs/product/REQUIREMENTS.md`
3. `process/docs/product/SCENARIOS.yaml`
4. `process/docs/product/TEST-MATRIX.md`
5. `process/docs/product/STORY-MAP.md`
6. `process/docs/product/MVP-SCOPE.md`
7. `process/docs/product/RELEASE-SLICES.md`
8. `process/docs/product/BACKLOG.md`

### Discussion 与自动检查

1. `process/discussions/CP2-CR052-VNEXT-R2-SCENARIO-DISCUSSION-LOG.md`
2. `process/checks/CP2-CR052-VNEXT-R2-DISCUSSION-CHECKPOINT.json`
3. `process/checks/CP1-CR052-USE-CASE-COMPLETENESS-VNEXT-R2.result.json`
4. `process/checks/CP1-CR052-USE-CASE-COMPLETENESS-VNEXT-R2.summary.md`
5. `process/checks/CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2.result.json`
6. `process/checks/CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2.summary.md`
7. `process/handoffs/CR052-CP2-VNEXT-REFRAME-META-PM-RETURN-SUMMARY.md`

## 校验结果

| 校验 | 结果 | 证据摘要 |
|---|---|---|
| Workspace routing | PASS | `meta-flow workspace check --project-root .`：`process_link_health=ok` |
| SCENARIOS YAML / discussion JSON 解析 | PASS | 通过 `uv run --with pyyaml --python 3.11` 解析；总 TC=117，TC-VNEXT=20，ID 唯一 |
| vNext 追溯 | PASS | UC=6、功能 REQ=21、约束 REQ=6、NFR=5、TC=20、候选 Story=6；REQ-VNEXT 未覆盖数=0 |
| CP1 result schema/contract | PASS | `meta-flow cp result-check` 返回 `CP Result Check: OK` |
| CP2 result schema/contract | PASS | `meta-flow cp result-check` 返回 `CP Result Check: OK` |
| 历史保留 | PASS | 八份产品文档均追加修订记录，旧 ID 未删除并有状态映射 |
| 单写边界 | PASS | 未修改源码、测试、`process/changes/*`、`process/state/*`、`process/checkpoints/*`、`process/context/*` 或其他项目路径 |

说明：基础 Python 环境未预装 PyYAML，首次解析尝试因此不可用；随后只使用临时 `uv --with pyyaml` 环境完成解析，未修改项目依赖或锁文件。该环境差异不是产品阻断项。

## Read Expansion

本 handoff 的 `read_profile=minimal`、`max_source_files=8`。为满足“八份正式产品对象全部增量更新、保留旧 ID/修订并完成跨文档追溯”的强制退出条件，执行了必要的深度评审扩读：在 capsule、summary、REQUEST、轻量 State 和三个直接 allowed 产品基线之外，读取了五个 `read_if_needed` 产品基线；任务源文件总数为 12，超过上限 4 个。

扩读原因逐项如下：

| 扩读对象 | 触发原因 |
|---|---|
| `process/docs/product/SCENARIOS.yaml` | 必须增量编写 `TC-VNEXT-*`、保持旧 TC 和统计一致 |
| `process/docs/product/TEST-MATRIX.md` | 必须建立 UC/REQ/TC/候选 Story/切片追溯并证明 P0/P1 无缺口 |
| `process/docs/product/STORY-MAP.md` | 必须将旧 MR 候选规划显式 supersede，并形成 vNext 候选 Story/活动 |
| `process/docs/product/RELEASE-SLICES.md` | 必须固化 A-D 当前推荐顺序并保留旧切片历史 |
| `process/docs/product/BACKLOG.md` | 必须把拆史、转换、自动 merge、批量迁移和 publication 隔离为 deferred/non-authorized |

完整任务源读取集合为：capsule、vNext summary、REQUEST、`STATE.current.json` 和八份产品基线。handoff 与已触发 Skill 的 `SKILL.md` 作为执行协议读取，不计入产品事实源。没有读取 `process/changes/CR-052.md`、`process/STATE.md`、旧 Migration Readiness 人工 checkpoint、`process/archive/**`、`process/stories/**` 或完整会话 transcript。根据 handoff 的单写限制，没有修改 capsule 的 `read_expansion_log`，也没有写入 `READ-EXPANSION-LEDGER`；扩读事实集中记录在本节供 host-orchestrator 审计。

## 阻断项与剩余风险

- 产品建模阻断项：无。
- 工具契约阻断项：无；CP1/CP2 两份 result 均已通过检查器。
- 剩余门控：CP2 人工决策尚未完成；在 CP2 `approved` 前，`ready_for_design=false`，不得推进 HLD、Story、LLD 或实现。
- 试点风险：G0/G1 上限、main-only publisher 吞吐、30 天观察期仍是推荐默认，只有人工批准和后续试点数据才能将其升格为正式基线。

## Host Orchestrator 后续动作

1. 读取本 return summary 和两份 machine result，去重汇总 DQ-VNEXT-01..05。
2. 生成/更新 VNEXT-R2 的 CP2 Decision Brief 与人工 checkpoint，并向用户说明 `approve` 的授权边界；这些操作不在本 meta-pm 单写范围内。
3. 人工决策后由 host-orchestrator 回写 `STATE.current.json` / `STATE.md`、handoff dispatch 完成信息和相应 checkpoint/agent/gate ledger；本 Agent 未代写这些状态。
4. 只有 CP2 正式批准后，才可路由 solution-design；若用户修改任一推荐值，应先按 CR 产品澄清规则回写正式基线再继续。
