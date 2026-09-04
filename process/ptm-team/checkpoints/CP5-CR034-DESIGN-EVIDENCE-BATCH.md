---
checkpoint_id: "CP5-CR034-DESIGN-EVIDENCE-BATCH"
checkpoint_name: "CR-034 11 文档同步点设计证据批量确认"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-30T15:45:00+08:00"
reviewed_by: "zhaohaibo"
reviewed_at: "2026-07-30T16:05:00+08:00"
auto_check_result: "process/checks/CP2-CR034.result.json"
cp2_result_ref: "process/checks/CP2-CR034.result.json"
target:
  phase: "story-planning"
  cr_id: "CR-034"
lld_policy: "batch-lld（CR-034 为文档同步 CR，11 个同质文档同步点，shared_contract=对齐已验证实现）"
---

# CP5 人工审查 - CR-034 11 文档同步点设计证据批量确认

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR034.result.json` | PASS | 0 | 范围基线 + 3 DQ approved（DQ-034-01 修订 topology/ 目录 / DQ-034-02 选项 A / DQ-034-03 不扩范围） |
| route plan | PASS | 0 | CP3/CP4 不 applies（无新设计/无 Story 拆解强制）；CP5 applies（实现设计证据） |
| 设计证据覆盖 | PASS | 0 | 11 个文档同步点（A1-A5 + 缺口 1-6）全部含 before/after/验证；A4/A5 来自 CP5 交叉评审补充（2026-07-30，不改变范围基线） |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 9 个文档同步点的精确修改方案（before/after），授权进入 CP6 文档编辑。 |
| 推荐动作 | `approve`：接受 9 个点的设计证据（修复方案），授权 CP6 实现。 |
| approve 后会发生什么 | host-orchestrator 自动推进 CP6（按设计证据编辑 4 规范文档 + install.py + 模板 + .gitignore）-> CP7 验证（dry-run + grep + install 一致性）-> CP8 交付。 |
| approve 不授权什么 | 不授权真实设备 `--execute`、case_runner/op_mapper 代码改动、24 用例 md 改动、外部写入/生产/发布。 |
| 不确认会阻塞什么 | 阻塞 CP6 文档编辑。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/checkpoints/CP2-CR034-REQUIREMENT-BASELINE.md` + 本文件 |
| read_profile | compact |
| 默认读取策略 | CP2 checklist + 本文件设计证据；不读取完整 case_runner.py/op_mapper.py 全文（行号已核实引用） |
| 关键数字 | 11 文档同步点 / 5 文件（ptm-te.md + 2 SKILL.md + install.py + 模板）+ .gitignore / 0 runtime |
| 全文档读取 | 默认不读完整 case_runner.py；CR-034.md 已按 read_expansion_refs 扩展（CP0/CP1/CP2） |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP2 approved DQ | `process/checks/CP2-CR034.result.json` | scanned | 3 | 0 | 已 approved，CP5 仅落实为设计证据 |
| 9 文档同步点 | 本文件设计证据 | scanned | 9 | 0 | 修复文本属 agent 默认处理，无新决策 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 0 | CP2 已闭环全部决策项 |
| 高风险策略确认 | 0 | 本 CR 低风险文档同步，无 runtime/credential/production-write |
| agent 默认处理 | 11 | 11 个点的精确修复文本（含 A4/A5 交叉评审补充） |
| 仅审计记录 | 1 | O-034-02 A1 L507 v1.4 修订记录处理（保留历史 + 补 v1.7 说明） |

### 待人工决策清单

本轮待人工决策项: 0

零决策原因：CP2 已闭环全部 3 项决策（DQ-034-01/02/03 approved）。CP5 仅落实为 9 个文档同步点的精确设计证据（before/after），无新范围/方案/授权决策。9 个点的修复文本属 agent 默认处理（本文件设计证据已明确 before/after，approve 即授权 CP6 按设计证据实现）。

## 9 个文档同步点设计证据

> design_evidence_type=batch-lld；homogeneous_story_pattern=规范文档对齐已验证实现；shared_contract=case_runner.py/op_mapper.py（不改实现）；risk_level=low。

### A1. ptm-te.md L456 注记滞后 + L507 v1.4 修订记录

- **文件:行**：`agents/ptm-te.md:456` + `:507`
- **before（L456）**：`> 注：...当前编排流程 [4] 的 op_mapper 只映射 fw_*，遇到 tg_* op 的调度分支待补充（见下方"待办：tg_* 编排分支"）。`
- **before（L507 v1.4 记录）**：`...tg_* 编排分支待补充，见"待办"小节。`
- **after（L456）**：改写为`> 注：trex-traffic 承载 tg_* 流量类 op，policy-route-execution 承载 fw_* 设备配置类 op，两者 op_id 前缀不重叠。编排流程 [4] 的 tg_* 调度分支已于 v1.5 实现（见下方"tg_* 编排分支（v1.5 已实现）"章节）：tg_* 与 fw_* 同走 op_mapper.execute_op，无需前缀分发。`
- **after（L507 v1.4 记录）**：保留历史，但在 v1.7 修订记录补说明（O-034-02：v1.4 记录的"待补充"已于 v1.5 完成，见 v1.5/v1.7 记录）。
- **验证**：`grep -n "待补充\|待办" agents/ptm-te.md` -> 仅 L507 历史记录保留（含 v1.7 说明），L456 无"待补充"。

### A2. case-execution/SKILL.md L37 命令示例文件名

- **文件:行**：`skills/case-execution/SKILL.md:37`
- **before**：`--case-file cases/IPv4策略路由/配置管理/策略配置/PC-M4-01-01-创建策略路由-有效参数.md`
- **after**：`--case-file cases/IPv4策略路由/配置管理/策略配置/PC-M1-01-01.md`（ST-EX-13 后 `<编号>.md`，编号格式 `PC-M1-01-01`）
- **验证**：`grep -n "PC-M4-01-01-创建" skills/case-execution/SKILL.md` -> 0 命中。

### A3. case-execution/SKILL.md L39/45/51 env-file 路径（与缺口 2 统一）

- **文件:行**：`skills/case-execution/SKILL.md:38/39/44/45/50/51`
- **before**：`--devices-yaml configs/devices.yaml` + `--env-file configs/topology.yaml`（L38/39/44/45/50/51 共 6 处 `configs/`）
- **after**：`--devices-yaml topology/devices.yaml` + `--env-file topology/topology.yaml`（DQ-034-01 topology/ 目录）
- **验证**：`grep -n "configs/" skills/case-execution/SKILL.md` -> 0 命中（"已废弃"说明除外）。

### A4. case-execution/SKILL.md L280 Gotcha#1 EXPECTED_OP_COUNT 滞后（CP5 交叉评审遗漏 1）

- **文件:行**：`skills/case-execution/SKILL.md:280`
- **before**：`8. **EXPECTED_OP_COUNT=22**（Gotcha #1）：op_mapper 含 fw_logout 映射后 EXPECTED_OP_COUNT 从 21 改为 22，否则 validate_mapping_consistency 报错。`
- **after**：`8. **EXPECTED_OP_COUNT=23**（Gotcha #1）：op_mapper 含 fw_logout + fw_delete_object（PF-03）映射后 EXPECTED_OP_COUNT 从 22 改为 23，否则 validate_mapping_consistency 报错。`（代码层 `case_runner validate` 实际输出 "23 op_id 全覆盖，三表一致"）
- **验证**：`grep -n "EXPECTED_OP_COUNT=22" skills/case-execution/SKILL.md` -> 0；`case_runner validate` 输出 23 op_id。

### A5. ptm-te.md L262 fw_delete_object 覆盖矩阵 gap 滞后（CP5 交叉评审遗漏 3）

- **文件:行**：`agents/ptm-te.md:262`
- **before**：`| fw_config_object | object config | state_mutation | （空，inverse_op=fw_delete_object 安装版未暴露，见覆盖矩阵 gap） |`
- **after**：`| fw_config_object | object config | state_mutation | inverse_op:fw_delete_object（op_mapper 已映射 PF-03；ptm-atomic 安装版 1.6.0 已暴露 fw_delete_object，destructive/restore_snapshot） |`
- **核实**：`ptm-atomic list | grep fw_delete_object` -> fw_delete_object 1.6.0 destructive restore_snapshot（**已暴露**，"安装版未暴露" gap 滞后）。
- **验证**：`grep -n "安装版未暴露" agents/ptm-te.md` -> 0 命中。

### 缺口 1（阻断性）. install.py L474 规则 4 执行入口

- **文件:行**：`script/ptm_team/install.py:474`
- **before**：`4. **执行入口**：用例从 \`cases/upload/<特性名>特性测试用例.md\` 读取，不直接读 ptm-tde 的 \`ppdcs/delivery/\`。`
- **after**：`4. **执行入口**：用例从 \`cases/<协议族>/<功能域>/<子域>/<用例编号>.md\` 读取（\`--cases-dir\` 递归 glob，ADR-07 编号正则排序），不直接读 ptm-tde 的 \`ppdcs/delivery/\`。\`cases/upload/\` 已废弃（CR-033 ST-EX-13），仅作历史回退。`
- **验证**：`python -c "import ast; ast.parse(open('script/ptm_team/install.py').read())"` 语法 OK；`grep -n "cases/upload" script/ptm_team/install.py` -> 仅 ptm-te.md 引用（规则块无 cases/upload）；规则块 L474 与 ptm-te.md v1.7 入口一致。

### 缺口 2. 环境文件路径统一（DQ-034-01 topology/ 目录 + DQ-034-02 devices.yaml.example）

- **文件**：`skills/case-execution/SKILL.md`（L39/45/51，A3 已含）+ `agents/ptm-te.md` 工作目录章节 + `.gitignore` + `skills/case-execution/templates/topology.yaml.example`（重命名）+ `skills/case-execution/templates/devices.yaml.example`（新建）
- **before**：
  - SKILL.md `--env-file configs/topology.yaml`（configs/ 不存在）
  - 模板 `topology-link3.yaml.example`
  - 无 case-execution 专用 devices.yaml.example（device-management 用 device_groups，与 resolve_addresses 顶层结构脱节）
  - .gitignore 无 topology/ 条目
- **after**：
  - 环境文件归档：workspace `topology/topology.yaml` + `topology/devices.yaml`（实际文件，不入库）；`--env-file topology/topology.yaml` + `--devices-yaml topology/devices.yaml`
  - 模板重命名：`topology-link3.yaml.example` -> `topology.yaml.example`（内容不变，含 port_mapping/nodes/links + ${ENV.*} 9 类占位符说明）
  - 新建 `skills/case-execution/templates/devices.yaml.example`：顶层 `firewall`（host/username/password_env）+ `tg`（api_server）结构，对齐 `resolve_addresses`(case_runner.py:505-520)；device-management 保留 device_groups（多设备组管理，不动）
  - .gitignore 加 `topology/`（防御性忽略 workspace 环境文件目录，防止误入库）
  - ptm-te.md 工作目录章节同步：topology/ 目录约定 + .example 模板入库
- **验证**：`grep -rn "configs/topology\|configs/devices" agents/ skills/` -> 0；`ls skills/case-execution/templates/` 含 topology.yaml.example + devices.yaml.example；devices.yaml.example 顶层 firewall+tg 与 resolve_addresses 一致。

### 缺口 3. ptm-te.md [5] 结果判定四态

- **文件:行**：`agents/ptm-te.md:99-103`
- **before（L103）**：`- **判定规则**：status=success + error_type=NONE + Check 点匹配 -> PASS；否则 -> FAIL`（两态）
- **after（L99-103）**：改为四态说明：
  ```
  - **判定规则（四态，ST-EX-10，ISTQB）**：classify_result 按 envelope 判定单步状态：
    - status=success + error_type=NONE + Check 点匹配 -> PASS
    - 业务错误（_BUSINESS_ERROR_TYPES，如 STATE_MISMATCH）+ known_issue=true -> KNOWN_FAIL；known_issue=false -> FAIL
    - 执行异常（_EXEC_ERROR_TYPES，如 TIMEOUT/CONNECTION_ERROR）-> ERROR（known_issue 不豁免）
  - **聚合判定**：classify_overall 取最高严重度 ERROR > FAIL > KNOWN_FAIL > PASS；用例整体状态取所有 step 最高级。
  - 引用：case_runner.py:794 classify_result / :825 classify_overall
  ```
- **验证**：`grep -n "KNOWN_FAIL\|classify_overall\|ERROR" agents/ptm-te.md` -> [5] 含四态 + 聚合规则。

### 缺口 4. retry + known_issue 字段说明

- **文件:行**：`agents/ptm-te.md` PC 消费契约（L135-145 字段表）+ `skills/case-execution/SKILL.md` case_steps 约定（L211+ / L258-259 附近）
- **before**：PC 消费契约字段表仅 step_name/op_id/args/expected_result；case_steps 约定仅 warming_up/max_loss（L258-259），无 retry/known_issue。
- **after**：
  - ptm-te.md PC 消费契约字段表追加 2 行：
    - `retry` | `case_steps[].retry` | 可选。重试配置 `{interval, max_attempts, success_condition}`，由 apply_retry 轮询消费（case_runner.py:1030）。success_condition 正则解析（_eval_success_condition，禁 eval/exec）。适用 hitcount 轮询类用例。
    - `known_issue` | `case_steps[].known_issue` | 可选。`true` 标注已知缺陷，classify_result 将业务错误 FAIL 豁免为 KNOWN_FAIL（case_runner.py:818-822）；执行异常 ERROR 不豁免。
  - case-execution SKILL.md case_steps 约定表（L258-259 附近）追加 2 行：
    - `retry 轮询` | 无 | `retry: {interval: 2, max_attempts: 5, success_condition: "hitcount == 0"}`（apply_retry 轮询，success_condition 正则）
    - `已知缺陷标注` | 无 | `known_issue: true`（FAIL->KNOWN_FAIL，执行异常不豁免）
  - case_steps 示例补 retry/known_issue 用法（见缺口 6 示例或独立示例）
  - **遗漏 2（CP5 交叉评审）**：`skills/case-execution/SKILL.md:158-160` 相邻对象边界表三项"case_runner 预留调用点"（tag/keyword 过滤、warming_up/retry/四态、report.md 生成）-> "case_runner 已实现"（ST-EX-08/09/10/11/16 已于 CR-033 Wave 3 实现，非预留）
- **验证**：`grep -n "retry\|known_issue" agents/ptm-te.md skills/case-execution/SKILL.md` -> PC 契约 + case_steps 约定含两字段说明；`grep -n "预留调用点" skills/case-execution/SKILL.md` -> 0 命中。

### 缺口 5. policy-route/SKILL.md src_addr -> source_network

- **文件:行**：`skills/policy-route-execution/SKILL.md:56-58/74-76/230/282/289/297/348`
- **before**：
  - L56-58 三层命名对照表：`src_addr`/`dst_addr`/`next_hop`（PC args 列）
  - L74-76 args->flag 映射：`src_addr`/`dst_addr`/`next_hop`（args key 列）
  - L230："三层命名不一致：ptm-tde PC src_addr ≠ op yaml source_network ≠ CLI --source-network"
  - L282/289/297 示例 args：`{"src_addr":"10.0.0.0/24",...}`
  - L348 修订记录："本 SKILL 映射表示例仍含旧 src_addr 命名，留 follow-up 同步"
- **after**：
  - L56-58：PC args 列改 `source_network`/`dst_network`/`next_hop_ip`（CR-025 后 args=params，三层对照变两层：args=params vs CLI flag 连字符）
  - L74-76：args key 列改 `source_network`/`dst_network`/`next_hop_ip`
  - L230：更新为"CR-025 后 args key 已对齐 op yaml params（source_network/dst_network/next_hop_ip），仅 CLI flag 连字符差异化；op_mapper ARGS_TO_FLAGS 承载翻译，不假设同名"
  - L282/289/297：示例 args 改 `{"source_network":"10.0.0.0/24",...}`
  - L348：清理"留 follow-up"，追加 v1.3 修订记录（CR-034 同步 args key 对齐）
- **验证**：`grep -n "src_addr\|dst_addr\|\"next_hop\"" skills/policy-route-execution/SKILL.md` -> 0 命中（L348 历史修订记录引用除外）。

### 缺口 6. TG 五步链完整示例

- **文件:行**：`agents/ptm-te.md` case_steps 示例（L153+）或 `skills/case-execution/SKILL.md` case_steps 示例
- **before**：case_steps 示例仅纯 DUT 步骤（fw_config_policy_route），无含 TG 发流的完整示例。
- **after**：在 case-execution/SKILL.md case_steps 示例章节追加 1 个含 TG 五步链的完整示例：
  ```yaml
  # M3/M4 类用例：TG 发流 + DUT 配置 + 流量验证（五步链）
  - step_id: STEP-001
    step_name: 配置 TG 接口（port1/port2）
    target: tg
    op_id: tg_config_interface
    args:
      ports: ${ENV.tg.ports[port1,port2]}   # 聚合占位符
      ip: ${ENV.tg.port1.ip}
  - step_id: STEP-002
    step_name: 应用流量模板
    target: tg
    op_id: tg_apply_traffic_template
    args: {template: "ipv4_2flow"}
  - step_id: STEP-003
    step_name: 启动流量（ARP 预热）
    target: tg
    op_id: tg_start_traffic_stream
    warming_up: true            # ADR-06，引擎强制 post_op tg_stop_traffic_stream
    args: {duration: 10}
  - step_id: STEP-004
    step_name: 验证流量损失
    target: tg
    op_id: tg_verify_traffic_loss
    expected_result: "loss < max_loss"
    args: {max_loss: 0}         # 显式 max_loss（规则 10）
  - step_id: STEP-005
    step_name: 停止流量
    target: tg
    op_id: tg_stop_traffic_stream
  ```
  （含 warming_up: true + max_loss + ${ENV.tg.ports[port1,port2]} 聚合 + ${ENV.tg.port1.ip}）
  - **交叉引用（CP5 交叉评审小问题 1）**：ptm-te.md case_steps 示例（L153+）保留纯 DUT 为基准示例，TG 五步链完整示例见 case-execution/SKILL.md（避免 ptm-te.md 膨胀；agent 首读 ptm-te.md 找格式，需 TG 示例时跨到 SKILL.md）
- **验证**：`grep -n "tg_config_interface\|tg_start_traffic_stream\|warming_up" skills/case-execution/SKILL.md` -> 含 TG 五步链示例；ptm-te.md 含交叉引用说明。

## 量化验证检查点（CP7 前置）

1. `grep -rn "cases/upload\|atomic_op\|src_addr\|PC-M4-01-01-01\|target: DUT" agents/ skills/ script/` -> 0 残留（"已废弃"说明除外）
2. ptm-te.md [5] 含 PASS/FAIL/KNOWN_FAIL/ERROR + classify_overall
3. PC 契约 + case_steps 约定含 retry/known_issue
4. install.py L474 与 ptm-te.md v1.7 入口一致
5. SKILL.md env-file 路径 = topology/topology.yaml；devices.yaml.example 顶层 firewall+tg
6. dry-run case_runner 可解析新格式示例
7. case_runner validate 23 op_id 三表一致
8. 含 TG 五步链示例（warming_up + max_loss + 聚合占位符）
9. ptm-te.md [5] 四态文字与 case-execution/SKILL.md Gotcha#1/边界表不矛盾（交叉一致性，CP5 交叉评审风险）
10. SKILL.md Gotcha#1 EXPECTED_OP_COUNT=23（A4，代码 validate 实际 23）
11. ptm-te.md L262 fw_delete_object 无"安装版未暴露"gap（A5，CLI 1.6.0 已暴露）
12. SKILL.md 无"预留调用点"（缺口 4 扩展，CR-033 Wave 3 已实现）

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP2 范围/方案基线 approved | PASS | `process/checks/CP2-CR034.result.json` |
| 9 个点设计证据完整 | PASS | 本文件 |
| 不改实现确认 | PASS | 全部设计证据为文档/规则块文本同步 |

## Checklist

| # | 检查项 | 审查结果 | 证据 |
|---|---|---|---|
| 1 | 9 个点全部含 before/after/验证 | PASS | 本文件设计证据 |
| 2 | 缺口 1 阻断性（install.py L474）方案明确 | PASS | 缺口 1 设计证据 |
| 3 | 缺口 2 环境文件路径按 DQ-034-01/02 落实 | PASS | 缺口 2 设计证据 |
| 4 | 缺口 3 四态 + 聚合规则对齐 classify_result/overall | PASS | 缺口 3 设计证据 |
| 5 | 缺口 4 retry/known_issue 对齐 apply_retry/classify_result | PASS | 缺口 4 设计证据 |
| 6 | 缺口 5 args key 对齐 CR-025（source_network 等） | PASS | 缺口 5 设计证据 |
| 7 | 缺口 6 TG 五步链含 warming_up/max_loss/聚合占位符 | PASS | 缺口 6 设计证据 |
| 8 | 不改 case_runner/op_mapper 实现 | PASS | 全部为文档同步 |
| 9 | A4/A5 交叉评审遗漏已纳入（EXPECTED_OP_COUNT 22->23 + fw_delete_object gap） | PASS | A4/A5 设计证据 |
| 10 | 缺口 4 扩展含"预留->已实现"（遗漏 2） | PASS | 缺口 4 设计证据 |
| 11 | 缺口 6 含 ptm-te.md 交叉引用（小问题 1） | PASS | 缺口 6 设计证据 |
| 12 | CP7 交叉一致性检查点已加（#9-12） | PASS | 量化验证检查点 |

## Exit Criteria

| 条目 | 审查结果 | 证据 |
|---|---|---|
| 设计证据无未豁免缺陷 | PASS | 本文件 |
| 用户终验确认 | PASS | 本文件 | 用户 approved 2026-07-30T16:05，进入 CP6 |

## Deliverables

| 交付物 | 路径 |
|---|---|
| CP2 approved baseline | `process/checkpoints/CP2-CR034-REQUIREMENT-BASELINE.md` |
| CP5 设计证据 | 本文件 |
| CR formal doc | `process/changes/CR-034.md` |

## 修订记录

| 版本 | 日期 | 变更要点 |
|---|---|---|
| v1.0 | 2026-07-30 | 初始设计证据：9 文档同步点 before/after/验证 |
| v1.1 | 2026-07-30 | CP5 交叉评审补充：A4（EXPECTED_OP_COUNT 22->23）+ A5（fw_delete_object gap，CLI 1.6.0 已暴露）+ 缺口 4 扩展（预留->已实现）+ 缺口 6 交叉引用 + CP7 检查点 #9-12；9->11 个点 |
| v1.2 | 2026-07-30 | CP8 交叉评审追溯整改：CP7 验证发现的范围延伸（skills/README.md L51 + script/install.py L473，属缺口1/缺口2 自然延伸）回写 CR-034.md 范围扩展记录，追溯链完整（CP2 9项 -> CP5 11项 -> CP6 +2范围延伸 -> CP7 PASS） |

## 人工审查结果

**approved**（zhaohaibo，2026-07-30T16:05:00+08:00）

接受 11 个文档同步点设计证据（含 A4/A5 交叉评审补充：EXPECTED_OP_COUNT 22->23 + fw_delete_object gap 移除 + 预留->已实现 + 缺口6 交叉引用）。授权 CP6 实现 -> CP7 验证 -> CP8 交付。validation_mode=static-only + dry-run-only，无 runtime。

- 自动预检 PASS，设计证据完整
- approve 不授权：真实设备 `--execute`、case_runner/op_mapper 代码改动、24 用例 md 改动、外部写入/生产/发布

---

> **三个 exact 回复**：`approve` / `修改: <具体修改点>` / `reject`
> - `approve` = 接受 9 个点设计证据，授权 CP6 实现。
