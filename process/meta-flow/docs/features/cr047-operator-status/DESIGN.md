---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-05"
feature_name: "Operator and Status Convergence"
source_blueprint: "process/docs/design/CR047-WORKFLOW-TRUTH-BLUEPRINT.md"
source_hld: "process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md"
source_adr: "process/docs/design/CR047-WORKFLOW-TRUTH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR047-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-WT-006", "ST-WT-007"]
lld_policy_summary: "full-lld=2"
---

# Feature Design: Operator and Status Convergence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline / meta-se | 冻结三平台非交互入口、CR-046 protected-object firewall 与叠加式 release ceiling。 |

## 摘要与边界

| 项目 | 内容 |
|---|---|
| 目标 | 让 CI/Agent 能按文档执行 3 平台 dry-run，并让 CR-046 current projection 与正式 7/7 evidence 一致且不改写历史。 |
| 推荐方案 | 更新公开/交付文档与现有 installer/preflight；ST-WT-007 用对象身份 manifest 双验。 |
| 下游 Story | ST-WT-006、ST-WT-007 |
| 非目标 | 真实用户级安装、平台 receipt、独立 QA attestation、修改 CR-046 protected originals。 |

## 上游依据与现有位置

| 来源/区域 | 路径 | 消费内容 / 当前职责 |
|---|---|---|
| ADR | `ADR-WT-005/006` | 越界子 CR、CP7/CP8 evidence ceiling |
| Installer docs | `README.md`, `delivery/README.md`, `delivery/doc/USER-MANUAL.md` | project-scope non-TTY 示例 |
| Installer | `delivery/scripts/install.py`, `meta_flow/cli.py` | `--project-dir` 与 dry-run |
| CR-046 evidence | formal CR、7 CP7 results、evidence index、product matrix | current status projection 输入 |
| Tests | installer CLI tests + new status/firewall fixtures | dry-run 与 historical mutation rejection |

## 接口与对象身份契约

三平台命令固定为 project scope、full component、显式 `--project-dir .`、`--dry-run`。文档命令是派生入口，安装器行为仍以现有 CLI/platform contract 为准。

protected manifest 每项包含：

```text
source_cr object_type canonical_ref provenance_ref original_sha256 immutable allowed_operation
```

路径前缀不得作为唯一身份。CP6 pre-implementation 从 formal refs/evidence index/ledger attribution 捕获；CP6 完成首验；CP7 再验。

## 流程、失败与回退

1. ST-WT-007 生成 manifest；身份/hash 不确定即停止。
2. 只更新 current product/status projection、新 correction/index/hash 和 CR-047 自身证据。
3. CP6/CP7 分别复算，要求 `protected_original_hash_changes=0`。
4. ST-WT-006 在现有 preflight contract 完成后同步 README/USER-MANUAL 并执行三平台 dry-run。

任一 protected original 变化时 `BLOCKED/NEEDS_DESIGN_CLARIFICATION`，拆子 CR，不能在 CR-047 内“修回 hash”。文档/命令回退恢复上一公开入口；append-only correction 只能以新的 superseding correction 纠正。

## 结论上限与安全

- 有效 inline fallback 时 CP7 最高 PASS_WITH_RISK；无 fallback/独立 QA 则 BLOCKED。
- 任一继承风险 OPEN 时 CP8 最高 READY_WITH_RISK。
- 禁止 `independent-QA-verified`、`platform-attested`、`runtime-profile-verified` 声明。
- 不读取 credentials/runtime/SaaS，不修改 backup/quant-lab，不 commit/push。

## 测试与下游契约

- 3/3 dry-run、非 TTY missing-project-dir negative、README command extraction。
- 7/7 status mapping、stale CP2/0-implemented count=0、OPEN risk preserved。
- object identity ambiguity、prefix collision、hash mutation、append-only correction fixtures。
- LLD 必须分别冻结 ST-WT-006 文档/installer边界和 ST-WT-007 protected object source set。

## 风险与 Gotchas

- fixture 能拒绝伪造 receipt，不等于平台签发 receipt。
- 当前 status projection 可改，历史 chronology 原件不可改。
- README 示例通过只证明 dry-run，不证明真实安装或 custom agent runtime。
