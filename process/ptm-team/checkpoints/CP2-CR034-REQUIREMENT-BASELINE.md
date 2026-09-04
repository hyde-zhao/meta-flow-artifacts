---
checkpoint_id: "CP2-CR034-REQUIREMENT-BASELINE"
checkpoint_name: "CR-034 需求/范围基线门"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-30T15:25:00+08:00"
reviewed_by: "zhaohaibo"
reviewed_at: "2026-07-30T15:35:00+08:00"
auto_check_result: "process/checks/CP1-CR034.result.json"
cp1_result_ref: "process/checks/CP1-CR034.result.json"
cp0_result_ref: "process/checks/CP0-CR-034-BOOTSTRAP.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-034"
---

# CP2 人工审查 - CR-034 需求/范围基线门

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP0-CR-034-BOOTSTRAP.result.json` | PASS | 0 | bootstrap artifacts 就位；route plan 派生（CP0/CP1/CP2/CP5/CP6/CP7/CP8 applies，CP3/CP4 不 applies）；runtime/credential 未授权。 |
| `process/checks/CP1-CR034.result.json` | PASS | 0 | 9 个文档同步点（3 遗漏 + 6 缺口）结构化于 CR-034.md，每项含现状/问题/修复/状态；行号已独立核实；场景产物 N/A with reason（文档同步 follow-up）。 |
| Scenario Gray Areas | N/A | 0 | 文档同步 follow-up CR，无新用户场景灰区（fast-lane 等效，CP2 直接确认范围 + 缺口 2 方案）。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-034 需求/范围基线（9 个文档同步点）+ 缺口 2 环境文件路径方案（DQ-034-01/02/03），授权进入 CP5 设计证据 -> CP6 实现 -> CP7 验证。 |
| 推荐动作 | `approve`：接受 9 个文档同步点范围 + 3 个待决策项推荐方案（DQ-034-01 选项 A / DQ-034-02 选项 A / DQ-034-03 不扩范围）。 |
| approve 后会发生什么 | host-orchestrator 自动推进 CP5（9 个文档同步点设计证据批量确认），发起 CP5 人工门禁；CP5 通过后 CP6 文档编辑 + CP7 验证（dry-run + grep + install 一致性）；validation_mode=static-only + dry-run-only，无 runtime。 |
| approve 不授权什么 | 不授权真实设备 `--execute` 写操作（FU-01）、case_runner.py/op_mapper.py 代码逻辑改动、24 用例 md 改动（R-C-008）、PF-04 聚合占位符实现（FU-02）、外部写入/生产/发布。设计通过不等于运行授权。 |
| 不确认会阻塞什么 | 阻塞 CP5 设计证据与 CP6 实现；缺口 2 环境文件路径方案未定，文档与模板无法统一。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP1-CR034.context.json` |
| read_profile | compact |
| 默认读取策略 | CR-034.md 问题清单 + CP0/CP1 result + CR-034-prompt.md；不读取完整 case_runner.py / op_mapper.py 全文（行号已核实引用）。 |
| 关键数字 | 9 文档同步点（3 遗漏 + 6 缺口）/ 4 规范文档 + install.py / 2 模板 / 3 待决策项 / 0 runtime |
| 全文档读取 | 默认不读完整 case_runner.py；已按 read_expansion_refs 扩展读取 CR-034.md（CP0/CP1 审计边界）。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CR-034-prompt.md | 用户准备的需求规格 | scanned | 5 | 3 | DQ-034-01/02/03 转入待决策；topology 模板命名 + A1 修订记录处理归 agent 默认 |
| CR-033 follow-up 台账 | `CR-033-FOLLOW-UP-TRACKING-2026-07-30.md` | scanned | 1 | 1 | FU-04 并入 DQ-034-02 |
| case_runner.py | resolve_addresses/classify_result/apply_retry | scanned | 0 | 0 | 实现已验证，仅作行号核实 |
| install.py | L474 规则块 | scanned | 1 | 0 | 缺口 1 修复方案明确（agent 默认处理） |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 2 | DQ-034-01 环境文件路径方案 / DQ-034-02 devices.yaml.example schema 方向 |
| 高风险策略确认 | 0 | 本 CR 低风险文档同步，无 runtime/credential/production-write |
| agent 默认处理 | 1 | DQ-034-03 用例标注不扩范围（确认性）；A1-A3 + 缺口 1/3/4/5/6 修复文本（CP5 细化）；O-034-02 A1 L507 修订记录处理 |
| 仅审计记录 | 2 | CP0/CP1 自动预检 PASS；9 个文档同步点行号已核实 |

### 待人工决策清单

用户需决策事项：DQ-034-01（implementation）、DQ-034-02（implementation）、DQ-034-03（scope）。3 项中 DQ-034-01/02 阻塞 CP6（环境文件路径与 schema 未定无法统一文档），DQ-034-03 为确认性。approve 时一并接受推荐方案。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-034-01 | implementation | 缺口 2 环境文件路径：topology.yaml/devices.yaml 归档位置 + .gitignore + --env-file 传参 | 选项 A（用户修订）：workspace `topology/` 目录（`topology/topology.yaml`+`topology/devices.yaml`，.gitignore 忽略实际文件）+ `templates/*.example` 入库 + `--env-file topology/topology.yaml`；模板重命名 `topology-link3.yaml.example`->`topology.yaml.example` | B：workspace 根；C：`env/` 目录；D：保留 `input/` 命名 | 用户选 topology/ 专用目录，环境文件集中且语义清晰；B 最简但散落根；C 与 cases/ 同级约定不一致；D 命名语义不清 | 影响 SKILL.md L39/45/51 + ptm-te.md 工作目录 + .gitignore + 模板重命名；低风险 | 若 topology/ 目录与现有实践冲突，切 B |
| DQ-034-02 | implementation | 缺口 2 devices.yaml.example schema 方向（device_groups vs 顶层 firewall/tg） | 选项 A：case-execution 新建专用 `skills/case-execution/templates/devices.yaml.example`（顶层 `firewall`+`tg`，对齐 resolve_addresses L505-520）；device-management 保留 device_groups | B：改 device-management example 为顶层；C：保留 device_groups + 文档说明转换 | A 不破坏 device-management 多设备组设计，case-execution 模板与实现一致；B 破坏 device-management；C example 仍误导 | 影响 新增 case-execution/devices.yaml.example + SKILL.md 引用 + ptm-te.md；低风险 | 若用户要求两 skill schema 强制统一，切 B（需评估 device-management） |
| DQ-034-03 | scope | 24 用例 md 是否同步四态/retry/known_issue 标注 | 不扩范围：仅同步规范文档，用例标注属 FU-01 runtime 验证范畴 | 扩范围同步 24 用例标注 | 推荐不扩范围符合 R-C-008 + 范围控制；备选扩范围违背用例不入库 + 无 runtime 验证标注正确性 | 无（确认性记录）；风险：用例标注滞后但规范文档已就位 | 若用户要求用例标注同步，转 FU-01 runtime 验证或单独 CR |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP0 bootstrap | PASS | `process/checks/CP0-CR-034-BOOTSTRAP.result.json` |
| CP1 需求结构化 | PASS | `process/checks/CP1-CR034.result.json` |
| 问题清单行号核实 | PASS | CR-034.md（A1-A3 + 缺口 1-6 全部独立核实） |
| 五维度影响分析 | PASS | CR-034.md 五维度影响分析 |
| 待决策项识别 | PASS | DQ-034-01/02/03 |

## Checklist

| # | 检查项 | 审查结果 | 证据 |
|---|---|---|---|
| 1 | 9 个文档同步点范围明确（3 遗漏 + 6 缺口） | PASS | CR-034.md 问题清单 |
| 2 | 不改实现逻辑（case_runner/op_mapper） | PASS | CR-034.md 非目标 |
| 3 | 缺口 1 阻断性已识别（install.py L474 规则块） | PASS | CR-034.md 缺口 1 |
| 4 | 缺口 2 环境文件路径方案待决策（DQ-034-01/02） | PASS | 本文件待人工决策清单 |
| 5 | validation_mode=static-only + dry-run-only | PASS | CR-034.md 约束 |
| 6 | FU-04 并入缺口 2，FU-01/02/03 保持独立 | PASS | CR-034.md 关联 |
| 7 | 3 个待决策项含推荐 + 备选 + 回退 | PASS | DQ-034-01/02/03 |

## Exit Criteria

| 条目 | 审查结果 | 证据 |
|---|---|---|
| 自动预检无未豁免失败 | PASS | CP0/CP1 result |
| 用户终验确认 | PASS | 本文件 | 用户 approved（DQ-034-01 修订为 topology/ 目录方案），进入 CP5 |

## Deliverables

| 交付物 | 路径 |
|---|---|
| CR formal doc | `process/changes/CR-034.md` |
| route plan | `process/checks/CP0-CR034.route-plan.json` |
| CP0 result | `process/checks/CP0-CR-034-BOOTSTRAP.result.json` |
| CP1 result | `process/checks/CP1-CR034.result.json` |
| follow-up 台账 | `process/changes/CR-033-FOLLOW-UP-TRACKING-2026-07-30.md` |
| 源 prompt | `CR-034-prompt.md` |

## 修订记录

| 版本 | 日期 | 变更要点 |
|---|---|---|
| v1.0 | 2026-07-30 | 初始 Decision Brief：9 文档同步点 + 3 DQ（DQ-034-01/02/03） |

## 人工审查结果

**approved**（zhaohaibo，2026-07-30T15:35:00+08:00）

接受 9 个文档同步点范围（3 遗漏 + 6 缺口）+ 3 项决策：
- DQ-034-01：**修订为 `<项目根>/topology/` 目录方案**（`topology/topology.yaml` + `topology/devices.yaml`，.gitignore 忽略实际文件 + `templates/*.example` 入库 + `--env-file topology/topology.yaml` + 模板重命名 `topology-link3.yaml.example`->`topology.yaml.example`）
- DQ-034-02：选项 A（case-execution 新建专用 `devices.yaml.example`，顶层 firewall+tg，对齐 resolve_addresses；device-management 保留 device_groups）
- DQ-034-03：不扩范围（仅同步规范文档，用例标注属 FU-01）

授权推进 CP5 设计证据 -> CP6 实现 -> CP7 验证 -> CP8 交付。validation_mode=static-only + dry-run-only，无 runtime。

- CP0/CP1 自动预检 PASS
- approve 不授权：真实设备 `--execute` 写操作、case_runner/op_mapper 代码改动、24 用例 md 改动、PF-04 实现、外部写入/生产/发布

---

> **三个 exact 回复**：`approve` / `修改: <具体修改点>` / `reject`
> - `approve` = 接受全部 3 项推荐方案（DQ-034-01 选项 A / DQ-034-02 选项 A / DQ-034-03 不扩范围），授权推进 CP5。
> - 若仅需改某项：如 `修改: DQ-034-01 选选项 C`。
