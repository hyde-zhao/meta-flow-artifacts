---
doc_type: FOLLOW-UP-TRACKING
parent_cr: "CR-035"
created_at: "2026-07-31"
owner: "host-orchestrator"
status: "active"
---

# CR-035 Follow-up Tracking 台账（2026-07-31）

> CR-035（ptm-te TG 端口生命周期 config-once）已于 2026-07-31 CP8 approved 关闭（READY_WITH_RISK）。本台账记录 3 项后续 CR 候选，状态取值：candidate / active / blocked / spike_candidate / converted-to-spike / closed / cancelled / superseded。只有用户决定推进某项时才创建正式 CR 文件。

## 候选清单

| 候选 ID | 类型 | 描述 | 触发条件 | 状态 |
|---------|------|------|---------|------|
| CR-035-FU-01 | follow_up_tracking | runtime 端到端验证（真实 --execute acquire/config/release/skip 行为） | 需设备 + --execute 运行授权 | candidate |
| CR-035-FU-02 | follow_up_tracking | 3 预存测试 FAIL 修复（test_cr018_p2 x2 + test_install_mapping x1） | 独立 hotfix 或后续 CR | candidate |
| CR-035-FU-03 | follow_up_tracking | 多 TG 设备场景 per-device user_id 隔离（HLD §2.2 Out of Scope） | 多 TG 设备测试需求 | candidate |

## 详细说明

### CR-035-FU-01 runtime 端到端验证
- **背景**：CR-035 validation_mode=static-only + dry-run-only，真实 --execute 行为（acquire/config/release/skip）经 mock + dry-run 验证，未覆盖真实端口占用并发、trex-api 网络错误、force_config 真实重配。
- **范围**：真实设备 --execute 端到端验证 TG 端口生命周期 config-once 全流程。
- **触发**：需设备 + --execute 运行授权。
- **关闭条件**：真实环境验证 acquire-once/config-once skip/release-once 行为符合 HLD；17 个 TG 用例批量执行无重复配置 IP 干扰。
- **风险**：RISK-CR035-RUNTIME-NA（CP8 accepted，本 FU 为缓解措施）。

### CR-035-FU-02 预存测试 FAIL 修复
- **背景**：CR-035 全量回归发现 3 个预存 FAIL（git stash 验证非 CR-035 回归）：test_cr018_p2 x2（CR-018 P2 gate 测试）+ test_install_mapping（CR-033 引入 case-execution 时期望未更新，期望 4 skills 实际 5）。
- **范围**：修复 3 个预存 FAIL，不涉及 CR-035 代码。
- **触发**：独立 hotfix 或后续 CR。
- **关闭条件**：3 个测试 PASS；全量回归 0 FAIL。
- **风险**：RISK-CR035-PREEXISTING-TEST-FAIL（CP8 accepted，本 FU 为根治措施）。

### CR-035-FU-03 多 TG 设备 per-device user_id 隔离
- **背景**：CR-035 HLD §2.2 Out of Scope，假设单 TG 节点（env_topology.nodes.tg1）。多 TG 设备场景需 per-device user_id 隔离。
- **范围**：tg_user_id 生成支持多 TG 设备；config-once 标志可能需 per-device（AGA-3 备选 B）。
- **触发**：多 TG 设备测试需求。
- **关闭条件**：多 TG 设备场景 user_id 隔离；per-device config-once 标志设计 + 实现。
- **关联**：AGA-3（run 级单标志 vs per-port，CR-035 选 A 单标志）。

## 转换规则

- candidate -> active：用户决定推进，创建正式 CR 文件（如 CR-036）。
- candidate -> spike_candidate / converted-to-spike：需调研验证，转 Spike。
- candidate -> closed / cancelled：不再需要，关闭/取消。
- active -> closed：CR 完成 CP8 关闭。
- active -> superseded：被新 CR 取代。
