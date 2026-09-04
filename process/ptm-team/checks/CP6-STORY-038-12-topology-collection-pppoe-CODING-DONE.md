---
checkpoint: CP6
cr_id: CR-038
story_id: STORY-038-12
story_slug: topology-collection-pppoe
title: topology-collection Mock→SW 建模 — 编码完成检查
result: PASS
created_at: 2026-08-15
owner: meta-dev
---

# CP6-STORY-038-12-topology-collection-pppoe-CODING-DONE

## Entry Criteria

- [x] Story `status=dev-ready`，`dev_gate=cp5-approved` 已满足
- [x] 设计证据（waived）已确认，Story 卡片含 `## Waived 重访条件` 与 `## 实现证据`
- [x] 文件所有权 primary=`resource/network-topology/topology-collection.md`，无 shared/forbidden 冲突
- [x] 无 `depends_on`，W1 起点

## Checklist

| 检查项 | 结果 | 证据 |
|--------|:----:|------|
| 输出文件存在且非空 | PASS | `resource/network-topology/topology-collection.md` 修改 3 处（注释 / Mermaid 样式 / YAML pppoe 节点） |
| 文件名符合 kebab-case | PASS | `CP6-STORY-038-12-topology-collection-pppoe-CODING-DONE.md` |
| 未修改 REQUIREMENTS/HLD/ADR/蓝图三件套 | PASS | 仅改 topology-collection.md + Story 卡片实现证据 + DEV-LOG |
| 只改 PPPoE Server 建模标记（node_type=SW + role=pppoe-server） | PASS | `node_type: Mock`→`SW`，删 `sub_type: pppoe-server`，`role: pppoe-server` 保留 |
| 拓扑结构未变（不新增/删除 link、不改连接关系） | PASS | `git diff` 核对：link1/link2/link3 `endpoints` 与 tg1↔dut1↔pppoe↔tg1 连接关系完全未改 |
| 与 STORY-038-01 `node_type=SW` 显式匹配语义一致 | PASS | 参照 `test_topo_mapper_sw.py:110` 建模 `node_type='SW', role='pppoe-server'`（无 sub_type） |
| 其它 Mock 节点保持不变 | PASS | 本场景仅 tg1/dut1/pppoe 三节点，无其它 Mock 节点需保持 |
| 实现证据写回 Story 卡片 | PASS | `## 实现证据` 小节已追加 |
| Story 状态更新为 ready-for-verification | PASS | frontmatter `status: ready-for-verification` |
| DEV-LOG.md 已追加 | PASS | 见 DEV-LOG 追加条目 |

## Exit Criteria

- [x] 验收标准 1：`node3_dut1_tg1_pppoe_link4` 场景 PPPoE Server 节点为 `node_type=SW` + `role=pppoe-server`
- [x] 验收标准 2：与 STORY-038-01 topo_mapper 的 `node_type=SW` 显式匹配语义一致
- [x] 拓扑结构未变（link 数量、endpoints、连接关系不变）
- [x] 无阻塞项、无凭据、无新增代码/Fixture

## Deliverables

| 类型 | 路径 |
|------|------|
| 实现产物 | `resource/network-topology/topology-collection.md` |
| Story 实现证据 | `process/stories/STORY-038-12.md`（`## 实现证据`） |
| 编码完成检查 | `process/checks/CP6-STORY-038-12-topology-collection-pppoe-CODING-DONE.md` |
| 开发日志 | `DEV-LOG.md` |

## N/A 说明

- 实现执行证据（对象清单/契约映射/测试 fixture/最小切片/平台差异）：本 Story 为 waived 文档建模回填，
  无代码对象、无测试 fixture、无平台差异（纯 Markdown 拓扑文档），CP6 采用文档增量 + `git diff` 人工核对
  （validation_context=review + 集成 dry-run），IMPLEMENTATION.md 不强制（普通文档建模 Story）。
