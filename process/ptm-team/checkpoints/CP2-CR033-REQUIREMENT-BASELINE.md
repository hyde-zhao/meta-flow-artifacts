---
checkpoint_id: "CP2-CR033-REQUIREMENT-BASELINE"
checkpoint_name: "CR-033 需求/场景/范围基线门"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-28T09:45:00+08:00"
reviewed_by: "zhaohaibo"
reviewed_at: "2026-07-28T10:15:00+00:00"
auto_check_result: "process/checks/CP2-CR033.result.json"
cp1_result_ref: "process/checks/CP1-CR033.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-033"
---

# CP2 人工审查 - CR-033 需求/场景/范围基线门

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR033.result.json` | PASS | 0 | 4 画像 + 10 成功指标 + 9 使用场景 + 8 维覆盖 + 4 灰区 resolved + 6 Deferred Ideas。 |
| `process/checks/CP2-CR033.result.json` | PASS | 0 | 26 功能需求 + 8 约束 + 5 非功能 + 25 验证场景 + 14 Story 候选 + MVP 10 项 + 4 发布切片。`ready_for_design=true`。 |
| Scenario Gray Areas | PASS | 0 | 4 条 SGQ 用户可见确认交互完成（SGA-01 A / SGA-02 A / SGA-03 A / SGA-04 C）。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-033 需求/场景/范围基线，授权进入 solution-design（meta-se 产出 HLD/蓝图/Story 拆解）。 |
| 推荐动作 | `approve`：接受需求/场景/范围基线 + 3 个待决策项（DQ-01/02/03）的推荐方案。 |
| approve 后会发生什么 | 进入 solution-design 阶段，委托 meta-se 产出 BLUEPRINT/DOMAIN-MAP/DEPENDENCY-MAP/HLD/FEATURE-DESIGN-MATRIX，准备 CP3。 |
| approve 不授权什么 | 不授权真实设备 `--execute` 写操作、ptm-atomic 实际重装、exec_v4.py 代码改造执行、外部写入/生产操作/发布。设计通过不等于运行授权。 |
| 不确认会阻塞什么 | 阻塞 HLD 启动，CR-033 无法进入 solution-design。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/handoffs/CR-033-REQ-CLAR-meta-pm-RETURN-SUMMARY.md` |
| read_profile | compact |
| 默认读取策略 | CR-033.md 范围目标 + RETURN-SUMMARY 交还摘要 + CP1/CP2 result；不读取完整 USE-CASES/REQUIREMENTS 全文（需时按需扩展）。 |
| 关键数字 | 26 功能需求 / 25 验证场景 / 14 Story 候选 / 4 发布切片 / 4 SGQ / 6 Deferred |
| 全文档读取 | 默认不读取完整 USE-CASES/REQUIREMENTS 全文；需时按 full_doc_read_reason 扩展（capsule_missing/field_conflict/human_audit/deep_review）。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.current.json` | scanned | 0 | 0 | 无未决人工问题。 |
| SGQ discussion log | `CP2-SCENARIO-DISCUSSION-LOG-CR033.md` | scanned | 4 | 0 | 4 灰区已由用户确认（SGA-01..04）。 |
| CP1/CP2 自动预检 | `CP1/CP2-CR033.result.json` | scanned | 3 | 3 | DQ-01/02/03 转入待人工决策。 |
| RETURN-SUMMARY | meta-pm 交还摘要 | scanned | 3 | 3 | 同上，不阻塞 HLD。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 6 | DQ-01 runtime_authorization / DQ-02 fw_logout 降级 / DQ-03 known_issue 整改 / DQ-04 24用例迁移 / DQ-05 frontmatter 必填 / DQ-06 用例编号标识 |
| 高风险策略确认 | 0 | 所有真实运行行为均明确不在授权范围。 |
| agent 默认处理 | 4 | 4 条 SGQ 灰区已确认（SGA-01 A / SGA-02 A / SGA-03 A / SGA-04 C）。 |
| 仅审计记录 | 2 | CP1/CP2 自动预检 PASS；产物路径见 Deliverables。 |

### 待人工决策清单

用户需决策事项：DQ-01（runtime_authorization）、DQ-02（implementation）、DQ-03（scope）、DQ-04（scope）、DQ-05（implementation）、DQ-06（implementation）。6 项均不阻塞 HLD，approve 时一并接受推荐方案。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-01 | runtime_authorization | case_runner --execute 模式下真实设备写操作的 CP7 验证方式 | CP7 用 static review + dry-run 替代 runtime | 等 VALIDATION-ENV.yaml 授权后执行 runtime | 推荐 static review 不触发写操作安全；备选 runtime 更充分但需设备授权 | 影响 CP7 验证层；风险 runtime 路径未覆盖 | runtime 失败则降级 static review |
| DQ-02 | implementation | fw_logout op 在 ptm-atomic 安装版是否暴露 | 安装前 ptm-atomic show fw_logout 验证；未暴露降级清理 session 文件 | 强制 ptm-atomic 升级暴露 fw_logout | 推荐降级保证可用性；备选升级超 CR-033 范围 | 影响 fw_logout op 完整度；风险登出不彻底 | 降级为 session 文件清理 |
| DQ-03 | scope | 24 用例 md 的 known_issue 标注是否纳入 CR-033 | 纳入 R-F-021 ARP 整改时同步检查 known_issue 标注 | 单独 CR 处理 known_issue 标注 | 推荐同步整改效率高；备选单独 CR 增协调成本 | 影响 24 用例整改量；风险标注不完整致误判 | 只做 ARP 整改，known_issue 进 BACKLOG |
| DQ-04 | scope | 24 用例目录迁移是否全部纳入 CR-033 范围 | 全部纳入 R-F-021 在 M4 一次性完成 | 分批迁移先 P0 后续迭代 | 推荐一次性完成避免半迁移；备选分批降低单次工作量 | 影响 M4 工作量；风险半迁移状态混乱 | P3 用例迁移进 BACKLOG |
| DQ-05 | implementation | frontmatter 16 列哪些必填哪些可选 | 8 必填（编号/名称/三~五级目录/级别/类型/是否自动化）+ 8 可选 | 全部必填 | 推荐 8+8 平衡完整性和工作量；备选全部必填但信息缺失 | 影响 24 用例 frontmatter 补全量；风险可选列缺失 | 缺失列填 N/A 或 TBD |
| DQ-06 | implementation | 用例名称连字符与文件名分隔符冲突 | frontmatter 用例编号为唯一标识，文件名按编号前缀正则匹配 | 下划线替代连字符做名称内部分隔 | 推荐编号为唯一标识不改命名习惯；备选改命名增迁移成本 | 影响 case_runner 文件名解析；风险解析失败 | 解析失败报错让用户手动指定 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP1 用户场景完备门 | PASS | `process/checks/CP1-CR033.result.json` |
| Scenario Gray Areas 已处理 | PASS | 4 条 SGQ，`process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR033.md` |
| USE-CASES/REQUIREMENTS/SCENARIOS/TEST-MATRIX 产出 | PASS | `docs/product/*-PTM-TE-EXEC.*` |
| STORY-MAP/MVP-SCOPE/RELEASE-SLICES/BACKLOG 产出 | PASS | 同上 |
| ready_for_design | true | CP2 result |

## Checklist

| # | 检查项 | 审查结果 | 证据 |
|---|---|---|---|
| 1 | 需求覆盖三需求原始请求（TG建模/执行引擎/12条改进） | PASS | REQUIREMENTS-PTM-TE-EXEC.md 26 功能需求 |
| 2 | 场景覆盖正向/负向/边界/权限/失败恢复/预检 | PASS | SCENARIOS-PTM-TE-EXEC.yaml 25 场景 |
| 3 | 4 条 SGQ 灰区用户确认 | PASS | discussion log SGQ-01..04 |
| 4 | 12 条改进全量映射到需求/Story | PASS | REQUIREMENTS + STORY-MAP |
| 5 | MVP In/Out/Deferred 明确 | PASS | MVP-SCOPE 10 In / 8 Out / 13 Deferred |
| 6 | 发布切片与 P0-P3 一致 | PASS | RELEASE-SLICES 4 切片 |
| 7 | 3 个待决策项已识别且不阻塞 HLD | PASS | DQ-01/02/03 |
| 8 | 跨仓库路由已明确（SGA-02 A） | PASS | ptm-team canonical + install.py 回填 |

## Exit Criteria

| 条目 | 审查结果 | 证据 |
|---|---|---|
| 自动预检无未豁免失败 | PASS | CP1/CP2 result |
| 用户终验确认 | PASS | 本文件 | 用户已 approve v1.1（2026-07-28），进入 solution-design |

## Deliverables

| 交付物 | 路径 |
|---|---|
| USE-CASES | `docs/product/USE-CASES-PTM-TE-EXEC.md` |
| REQUIREMENTS | `docs/product/REQUIREMENTS-PTM-TE-EXEC.md` |
| SCENARIOS | `docs/product/SCENARIOS-PTM-TE-EXEC.yaml` |
| TEST-MATRIX | `docs/product/TEST-MATRIX-PTM-TE-EXEC.md` |
| STORY-MAP | `docs/product/STORY-MAP-PTM-TE-EXEC.md` |
| MVP-SCOPE | `docs/product/MVP-SCOPE-PTM-TE-EXEC.md` |
| RELEASE-SLICES | `docs/product/RELEASE-SLICES-PTM-TE-EXEC.md` |
| BACKLOG | `docs/product/BACKLOG-PTM-TE-EXEC.md` |
| SGQ 讨论日志 | `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR033.md` |
| CP2 Decision Brief 输入 | `process/handoffs/CR-033-REQ-CLAR-meta-pm-RETURN-SUMMARY.md` |

## 修订记录

| 版本 | 日期 | 变更要点 |
|---|---|---|
| v1.0 | 2026-07-28 | 初始 Decision Brief：4 SGQ + 3 DQ |
| v1.1 | 2026-07-28 | 范围修改：执行入口改为单用例/按目录/按标签或关键字；用例目录 cases/功能/子模块/用例组/；frontmatter 16 列 + case_steps YAML；用例命名 <编号>-<名称>.md；新增 DQ-04/05/06，SGQ-05 |
| v1.2 | 2026-07-28 | meta-se CP3 评审反馈 P3 落实：验证场景数 20->25（与 SCENARIOS-PTM-TE-EXEC.yaml 25 场景一致） |
| v1.3 | 2026-07-28 | host-orchestrator 事实纠正：功能需求数 21->26（与 REQUIREMENTS v1.1 的 R-F-001..026 及 CP2 result.json 一致，v1.1 漏改的 v1.0 残留） |

## 人工审查结果

**approve**（zhaohaibo，2026-07-28T10:15:00+00:00）

接受 v1.1 需求/场景/范围基线（含 5 点范围修改：执行入口/目录结构/frontmatter 16列/命名/标签）及 DQ-01..06 推荐方案，进入 solution-design。

- CP1/CP2 自动预检 PASS，ready_for_design=true
- approve 不授权：真实设备 `--execute` 写操作、ptm-atomic 重装、外部写入/生产/发布
