---
doc_type: FEATURE-DESIGN-MATRIX
id: FEATURE-DESIGN-MATRIX-CR-038
cr_id: CR-038
version: "0.2"
status: active
created_at: "2026-08-15"
updated_at: "2026-08-15"
owner: meta-se（story-planning）
source_hld: docs/design/HLD-CR-038.md
---

# Feature 设计矩阵 — CR-038（PPPoE 链路规划能力）

> 状态：active（CP3 已 approve 方案 A + ADR-CR038-01~05）。本矩阵为 Story `feature_design_refs` 与 `lld_policy` 的机器真相源，Story 卡片与 `process/DEVELOPMENT-PLAN-CR-038.yaml` 派生引用本矩阵。

## 1. Feature 适用性判定

| Feature ID | 名称 | 来源蓝图/HLD | 适用性 | 触发原因 | 产物路径 |
|---|---|---|---|---|---|
| F-CR038-E1 | SW 映射与台账 | BLUEPRINT §2 / HLD §10.1 | **required** | core-engine（topo_mapper 回溯改动 + reserved 互斥）+ shared-story-boundary（topo_mapper/pool_merge 台账互斥） | `docs/features/cr038-sw-mapping/DESIGN.md` / `TEST-PLAN.md` / `TASKS.md` |
| F-CR038-E2 | PPPoE 配置 | BLUEPRINT §2 / HLD §10.4 | **required** | external-interface（H3C telnet）+ security（cipher 密文）+ data-model（pppoe_server 块） | `docs/features/cr038-pppoe-config/DESIGN.md` / `TEST-PLAN.md` / `TASKS.md` |
| F-CR038-E3 | 环回 env-file | BLUEPRINT §2 / HLD §10.3 | **required** | cross-module-contract（exporter/case_runner 契约）+ data-model（interface_kind/trex_instance）+ shared-story-boundary（exporter/pool_merge/physical_pool 多 Story） | `docs/features/cr038-loopback-envfile/DESIGN.md` / `TEST-PLAN.md` / `TASKS.md` |
| F-CR038-E4 | SW 设备管理 | BLUEPRINT §2 / HLD §10.6 | **waived** | 模板/参考文档回填，模式既有 | N/A |
| F-CR038-E5 | 验证与集成 | BLUEPRINT §2 / HLD §10.6 | **waived** | 编排为 prompt 级集成（沿用 CR-037），验证由 meta-qa CP7 承担，建模为文档回填 | N/A |

> 注：E1 触发原因在本版从 HLD §17 的「data-model（interface_kind 扩展）」修正为「core-engine + shared-story-boundary」——interface_kind 扩展实际归属 E3（STORY-038-13 topology_model/parser），E1 的 topo_mapper 改动为回溯算法 + 台账互斥，不涉及数据模型字段变更。此修正不影响 HLD 推荐方案与 5 条 ADR，属 CP3 后落地映射细化（HLD 文件本身不修改）。

## 2. waived 理由与重访条件

| Feature | waived 理由 | 影响范围 | 重访条件 |
|---|---|---|---|
| F-CR038-E4 | devices.yaml sw 块 + device-reference SW 别名是模板/参考文档增量，无跨模块数据契约 | device-management 模板/参考 | 若 SW 设备块需要新 schema 校验逻辑（非模板）则恢复 required |
| F-CR038-E5 | ptm-te.md 编排沿用 CR-037 编排模式；trex 验证由 meta-qa 在 CP7 生成 TEST-STRATEGY；topology-collection 是文档建模 | agents/ptm-te.md + trex-traffic + 建模文档 | 若 PPPoE 编排需要新的跨 skill 状态机/契约则恢复 required |

## 3. Story 级 feature_design_refs + lld_policy 判定

| Story | Feature | feature_design_refs | lld_policy.required_level | 触发原因 | 依据 |
|---|---|---|---|---|---|
| STORY-038-01 | E1 | cr038-sw-mapping/DESIGN.md + TEST-PLAN.md + TASKS.md | full-lld | core-engine + shared-story-boundary | 显式 SW 匹配 + reserved 互斥 + role 传递，R-F-001/002/018 |
| STORY-038-02 | E1 | cr038-sw-mapping/DESIGN.md | technical-note | shared-story-boundary | SW 归并复用 _merge_sw_pc，R-F-011 |
| STORY-038-03 | E2 | cr038-pppoe-config/DESIGN.md + TEST-PLAN.md + TASKS.md | full-lld | external-interface + security + data-model | 三厂商命令 + pppoe_server schema + cipher 密文，R-F-003/004/019 |
| STORY-038-04 | E2 | cr038-pppoe-config/DESIGN.md | technical-note | external-interface | 复用 configure_* 下发模式，R-F-003 |
| STORY-038-05 | E2 | cr038-pppoe-config/DESIGN.md | full-lld | external-interface | ptm-atomic op 缺失三选一闭环，R-F-005，RA-038-001 |
| STORY-038-06 | E3 | cr038-loopback-envfile/DESIGN.md + TEST-PLAN.md + TASKS.md | full-lld | cross-module-contract + data-model | MVP 放宽 + links 校验 + SW nodes，R-F-006/007 |
| STORY-038-07 | E3 | cr038-loopback-envfile/DESIGN.md | technical-note | cross-module-contract | case_runner 兼容 SW 不破坏 tg1/dut1，R-F-008 |
| STORY-038-08 | E4 | （waived） | waived | — | 模板回填，R-F-009 |
| STORY-038-09 | E4 | （waived） | waived | — | 参考文档回填，R-F-010 |
| STORY-038-10 | E5 | （waived） | technical-note | — | trex 发流验证（meta-qa CP7 承担），R-F-012 |
| STORY-038-11 | E5 | （waived） | technical-note | — | prompt 级编排集成 |
| STORY-038-12 | E5 | （waived） | waived | — | 文档建模回填 |
| STORY-038-13 | E3 | cr038-loopback-envfile/DESIGN.md + TEST-PLAN.md + TASKS.md | full-lld | data-model + cross-module-contract + shared-story-boundary | 多实例 TRex + interface_kind 跨 5 文件，R-F-013~017 |

## 4. 需尽早确认的关键决策项

> CP2 已锁定 7 项方向性决策；CP3 已 approve 5 条 ADR（ADR-CR038-01~05）。本矩阵的 Feature 级关键决策均为实现层，已在 Feature DESIGN 中确定性定义，不新增用户决策项。

| 决策 ID | 类型 | 问题 | 推荐 | 备选 | 影响/风险 | 回退/切换 | 已落地 |
|---|---|---|---|---|---|---|---|
| FD-CR038-01 | implementation | explicit_sw_reserved 互斥粒度 | 整体剔除显式 SW 候选集 | 仅剔除已分配 SW | 显式 SW 失败时保守保留候选 | 需透传复用时切换 | E1 DESIGN §2 |
| FD-CR038-02 | implementation | trex_instances 合并方向 | pool 侧为准补 devices 侧 | devices 侧为准 | 台账同源一致性 | 实例改 devices 唯一源时切换 | E3 DESIGN §5 |
| FD-CR038-03 | implementation | pppoe_server 数据源 schema | physical_pool SW 节点块 | devices.yaml / 独立配置 | 同源一致 | 跨设备共享时切换 | E2 DESIGN §2 |
| FD-CR038-04 | implementation | interface_kind 默认 | physical | 显式必填 | 向后兼容 | 需严格校验时切换 | E3 DESIGN §4 |

## 5. 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-se | 初稿：3 required + 2 waived + Story lld_policy 判定 |
| 0.2 | 2026-08-15 | meta-se | CP3 通过后正式化（draft→active）；E1 触发原因修正为 core-engine + shared-story-boundary（interface_kind 归 E3）；§3 补 feature_design_refs 列；补「已落地」指向 Feature DESIGN 章节 |
