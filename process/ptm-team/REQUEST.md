---
request_id: "REQ-PTM-TE-EXEC-001"
version: "1.0"
status: "active"
submitted_at: "2026-07-28T08:35:00+08:00"
submitted_by: "user"
source: "user-direct-request + ptm-te-17-case-execution-retrospective"
target_artifact_type: "skill"
governance_mode: "conditional"
review_policy: "standard"
active_cr: "CR-033"
---

## 来源上下文

| 字段 | 当前值 | 说明 |
|---|---|---|
| `target_artifact_type` | `skill` | 交付对象为 ptm-te 工作流的 device-management 扩展与新建 case-execution skill |
| `governance_mode` | `conditional` | standard 工作流，CP2/CP3/CP5/CP8 关键人工门禁停留确认 |
| `review_policy` | `standard` | 多 Story + 架构 + 规则，不可 fast-lane |
| `active_cr` | `CR-033` | 统一目标包 parent CR |
| `source` | `user-direct-request +  retrospective` | 三个需求 + 17 用例执行复盘 12 条改进 |

## 用户目标

为 ptm-te 建立 TG 设备建模、case-execution 用例执行引擎与执行流程整改的统一能力，消除 exec_v4.py 硬编码与重装回退风险。

## 三个需求

1. **TG 设备建模**：设备管理技能添加 TG 类型设备（type:TG，含 ip 地址、串口地址，子类型 ixia-c/trex，硬件平台 EP/C236/J1900）。
2. **case-execution 用例执行引擎**：添加用例执行 skill 或 tool / 命令行，参考 `/home/hyde/projects/ptm-te` 的 exec_v4.py 执行脚本，可参考 pytest/robot 框架理念。
3. **12 条执行改进整改**：基于单用例到 17 用例全量执行踩坑复盘，按投入产出比排序的 12 条改进，全量分期落地。

## 核心范围（In Scope）

1. devices.yaml 新增 `tg` 块（type:TG + host + serial_url + sub_type + hardware_platform + ssh + api_server），6 种组合。
2. 新建 `skills/case-execution/`（SKILL.md + `scripts/case_runner.py` 兼 argparse CLI，op_mapper 模式），三入口零代码新增用例。
3. 全量 12 条改进 P0→P3 落地（含 fw_logout op、24 用例 ARP 预热整改、envelope 统一解析、失败诊断、结构化报告、known_issue 四态）。
4. 承接 CR-032 批次B（exec_v4.py 代码改造）。

## 明确排除（Out of Scope）

- 改 ptm-atomic CLI 本体
- 引入 pytest / robot / 外部 eval 框架
- 为 devices.yaml 引入 pydantic
- HTML 报告（进 BACKLOG）
- 改 traffic-skill / ngfw-install skill

## 交付预期

| 交付项 | 说明 |
|---|---|
| TG 设备模型 | devices.yaml tg 块 + SKILL.md + device-reference.md 6 组合对照 |
| case-execution skill | SKILL.md + case_runner.py，复用 op_mapper，硬编码全参数化 |
| 规则固化 | install.py ptm-te-workflow 规则块 ≥4 条新规则 |
| 12 条改进 | 12/12 落地，结果四态分级，重装不丢失 |

## 补充约束

1. 本 CR 为 standard 模式，CP2 需求/场景/范围基线门未通过前不得输出正式设计对象。
2. 跨仓库：exec_v4.py + 24 用例 md 在 `/home/hyde/projects/ptm-te/`（workspace），需明确目标路由。
3. 设备 `--execute` 写操作须先取得 runtime_authorization 确认（设计通过不等于运行授权）。

## 变更记录

| 版本 | 变更摘要 | 处理人 | 时间 |
|---|---|---|---|
| 1.0 | 建立 CR-033 三需求原始请求基线（TG建模 + case-execution + 12条改进）。 | host-orchestrator | 2026-07-28T08:35:00+08:00 |
