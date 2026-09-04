---
cr_id: "CR-033"
story_id: "STORY-EX-08"
story_name: "ARP 预热引擎（warming_up/post_op 引擎强制）"
story_slug: arp-warming-up-engine
lld_policy: full-lld
evidence_type: full-lld
design_evidence_type: full-lld
lld_policy_required_level: full-lld
version: "1.0"
created_at: "2026-07-28T16:00:00+08:00"
author: "meta-dev"
hld_ref: "docs/design/HLD-PTM-TE-EXEC.md v1.6 §12.1[5] / §8 场景3 / §14 RA-008 / §15 ADR-06"
adr_ref: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md ADR-06"
development_plan_ref: "process/DEVELOPMENT-PLAN.yaml ST-EX-08"
feature_design_ref: "docs/features/case-execution/DESIGN.md（FE-EX-02，CP3 后生成）"
tier: "M"
wave: "W3"
dependencies: ["ST-EX-04"]
dependency_type: "hard"
file_owners:
  - "skills/case-execution/scripts/case_runner.py#apply_warming_up"
shared_fragments: []
open_items: ["LCQ-ST-EX-08-01"]
status: "draft"
---

# LLD - ST-EX-08 ARP 预热引擎（warming_up/post_op 引擎强制）（CR-033）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-dev | CR-033 ST-EX-08 full-lld 初稿：apply_warming_up 引擎强制 + post_op 参数继承 + 双重保障（ADR-06）+ 14 章节完整 |

## ADR / HLD 追溯

| 来源 | 章节 | 决策/约束 | 本 LLD 落点 |
|---|---|---|---|
| ADR-06 | ARP 预热双重保障 | 规则 + 引擎双重保障（SGA-04=C）；规则被绕过时引擎兜底 | §3 接口设计 / §4 核心流程 / §8 回滚策略 |
| HLD §12.1[5] | warming_up -> 强制 post_op | warming_up:true 主 op 执行后强制 post_op（tg_stop_traffic_stream） | §4 核心流程 |
| HLD §8 场景3 | ARP 预热自动清理 | 即使 md 未写 post_op，自动补充，参数从主 op 继承（ports/txport/rxport/name） | §3 接口设计 / §4 核心流程 |
| HLD §14 RA-008 | 重装后 ARP 预热规则被绕过 | 引擎在 case_runner 内重装不丢失，兜底 | §8 回滚策略 |
| HLD §13.5 | 可维护性 | 引擎在 case_runner 内，install.py 规则块托管 | §7 实施步骤 |
| Gotcha #10 | install.py 规则块重装一致性 | 规则+引擎双重保障验证 | §6 测试设计 |
| SM-EX-07 | 24 用例 warming_up step 合规 | 24/24 | §6 测试设计 |

---

## 0. 概述

本 LLD 是 ST-EX-08 ARP 预热引擎（warming_up/post_op 引擎强制）的 full-lld 设计证据，属于 FE-EX-02 case-execution 引擎 Feature 的 Wave 3 引擎 Story，承载 case_runner.py ARP 预热双重保障的引擎侧。

**需求溯源**：ST-EX-08 承载 R-F-008（warming_up/post_op 引擎强制）、R-F-019（规则固化）、R-F-021（24 用例 ARP 预热校验）等功能需求；详细需求映射见 §1.1 需求来源。

**模块拆分**：case_runner.py 按职责拆分为 apply_warming_up（引擎强制 + post_op 参数继承）函数锚点（ST-EX-08 拥有）；run_case 主循环调用点（解析 case_steps 后、逐 step 执行前）；不改 op_mapper.py（ST-EX-03/07 拥有）/install.py（ST-EX-02 拥有）/24 用例 md（ST-EX-13 拥有）。

**代码结构**：详见 §2 文件影响范围。case_runner.py 新增 apply_warming_up 函数；只读依赖 op_mapper tg_start/stop_traffic_stream 映射（既有）+ install.py ARP 预热规则块（ST-EX-02，双重保障的规则侧）。

**技术细节**：引擎强制（§4.1，warming_up:true 的 step 后无论 md 是否写 post_op 自动补充 tg_stop_traffic_stream）；双重保障（§4.2，ADR-06，install.py 规则约束人 + case_runner 引擎约束机）；post_op 参数继承（§3.3，ports/txport/rxport/name 从主 op args 继承）；_warming_up_meta 三态（§3.4，auto/merged/explicit）等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试设计的 6 单元测试（UT-08-01..06）+ dry-run 24/24 warming_up 合规（SM-EX-07）全通过为准；CP7 验证采用 static-only + dry-run-only（DQ-01 推荐）。DoD 详见 §6.2/§6.3/§6.4 测试分层与 dry-run 集成测试。

## 1. 工程依据

### 1.1 需求来源

ST-EX-08 承载 CR-033 改进 #3 的"引擎"部分（HLD §16.1 追溯表：#3 ARP 预热 -> ST-EX-02 规则 / ST-EX-08 引擎 / ST-EX-13 24 用例校验）。

- 关联需求：R-F-008（warming_up/post_op 引擎强制）、R-F-019（规则固化）、R-F-021（24 用例 ARP 预热校验）
- 关联场景：SCN-EX-05（ARP 预热自动清理）、SCN-EX-19（重装一致性）
- 关联 UC：UC-EX-07（ARP 预热自动清理）

### 1.2 问题陈述

exec_v4.py 时代，ARP 预热规则（warming_up step 后必须执行 post_op 停流）只靠 install.py 规则块约束。重装后规则块可能被绕过或丢失（RA-008），导致 warming_up step 发起的 ARP 预热流量未停流，残留流量干扰后续用例。

### 1.3 设计目标

- 引擎强制：case_runner 执行 warming_up:true 的 step 后，无论 case_steps md 是否显式写 post_op，引擎自动补充并强制执行 tg_stop_traffic_stream
- 双重保障：install.py 规则块（ST-EX-02）+ case_runner 引擎（ST-EX-08），规则被绕过时引擎兜底
- 参数继承：post_op 参数从主 op（tg_start_traffic_stream）的 args 继承（ports/txport/rxport/name），不要求用例作者重复填写
- 可审计：result.json 记录 warming_up step 标记 auto_post_op=true，区分引擎补充与用例显式

### 1.4 量化验收

| 验收项 | 目标值 | 验证方式 |
|---|---|---|
| warming_up step 合规 | 24/24（SM-EX-07） | dry-run 扫描 24 用例全部 warming_up step 后均有强制 post_op |
| 引擎兜底 | md 漏写 post_op 时引擎自动补充 | 单元测试构造漏写 post_op 的 case_steps |
| 重装一致性 | 引擎在 case_runner 内，重装不丢失 | install -> uninstall -> install 循环验证 |

### 1.5 非目标

- 不实现 ARP 预热规则块文本（ST-EX-02 承载）
- 不改 op_mapper.py（ST-EX-08 仅在 case_runner.py）
- 不改 24 用例 md 的 warming_up 标注（ST-EX-13 承载）
- 不采集 TG 系统快照（R-C-006）

---

## 2. 文件影响范围

### 2.1 文件所有权（DEVELOPMENT-PLAN ST-EX-08）

| 文件 | 符号 | 操作 | 说明 |
|---|---|---|---|
| skills/case-execution/scripts/case_runner.py | apply_warming_up | 新增 | 引擎强制 + post_op 参数继承 |

### 2.2 只读依赖（不修改）

| 文件 | 来源 Story | 用途 |
|---|---|---|
| skills/policy-route-execution/scripts/op_mapper.py | ST-EX-03/04/07 | execute_op（tg_start/stop_traffic_stream 映射） |
| install.py ptm-te-workflow 规则块 | ST-EX-02 | ARP 预热规则（双重保障的"规则"侧） |
| ptm-te/cases/**/*.md | ST-EX-13 | 24 用例 warming_up 标注 |

### 2.3 不触碰文件

- op_mapper.py（ST-EX-03/07 拥有，ST-EX-08 不改）
- install.py（ST-EX-02 拥有）
- 24 用例 md（ST-EX-13 拥有）

---

## 3. 接口设计

### 3.1 apply_warming_up 函数签名

```python
def apply_warming_up(
    steps: List[dict],
    *,
    warming_up_op_id: str = "tg_start_traffic_stream",
    post_op_id: str = "tg_stop_traffic_stream",
) -> List[dict]:
    """扫描 case_steps，为 warming_up:true 的 step 强制补充 post_op。

    引擎强制逻辑（ADR-06 引擎侧）：
    1. 遍历 steps，识别 warming_up:true 且 op_id == warming_up_op_id 的 step
    2. 检查该 step 之后是否存在 op_id == post_op_id 的显式 post_op step
       - 存在且参数完整：标记该 post_op step 的 source="explicit"，不干预
       - 存在但参数缺失：用主 op 参数补全缺失字段，标记 source="merged"
       - 不存在：在主 op step 之后插入自动补充的 post_op step，标记 source="auto"
    3. post_op 参数继承：从主 op args 提取 ports/txport/rxport/name
    4. 返回补全后的 steps（不修改原列表，返回新列表）

    Args:
        steps: case_steps 解析后的 step dict 列表，每个 step 含
               step_name/op_id/args/warming_up(可选)/known_issue(可选)
        warming_up_op_id: 触发 ARP 预热的主 op（默认 tg_start_traffic_stream）
        post_op_id: 强制执行的清理 op（默认 tg_stop_traffic_stream）

    Returns:
        补全后的 steps 列表，新增/合并的 post_op step 含
        _warming_up_meta={source: "auto"|"merged"|"explicit",
                          trigger_step: <step_name>}
    """
```

### 3.2 调用契约

| 调用方 | 调用时机 | 输入 | 输出 | 后续衔接 |
|---|---|---|---|---|
| case_runner.run_case | 解析 case_steps 后、逐 step 执行前 | parsed steps | 补全后 steps | 逐 step 执行时按 _warming_up_meta 记录 auto_post_op |

### 3.3 post_op 参数继承表

| post_op 参数 | 来源（主 op tg_start_traffic_stream args） | 说明 |
|---|---|---|
| ports | args["ports"] | TG 端口列表（已 resolve_env_refs/resolve_step_refs 后的值） |
| txport | args["txport"] | 发送端口 |
| rxport | args["rxport"] | 接收端口 |
| name | args["name"] | 流名称（停流按 name 定位） |

继承的是主 op step **执行前已解析**的 args（resolve_env_refs/resolve_step_refs 在 execute_op 内完成，apply_warming_up 在 execute_op 前操作 parsed steps，故继承的是占位符未解析的原值；post_op 执行时由 execute_op 再次 resolve）。若主 op args 含 `${ENV.*}` / `${STEP-N.*}`，post_op 同样含相同占位符，由 execute_op 解析——保证 post_op 与主 op 解析到同一物理值。

### 3.4 _warming_up_meta 数据结构

```python
{
    "source": "auto" | "merged" | "explicit",
    "trigger_step": "<warming_up step 的 step_name>",
}
```

- `auto`：用例未写 post_op，引擎自动插入
- `merged`：用例写了 post_op 但参数缺失，引擎补全缺失字段
- `explicit`：用例写了完整 post_op，引擎不干预（仅标记）

该 meta 在 result.json 的 step 记录中输出为 `auto_post_op: true`（source=auto/merged）或 `auto_post_op: false`（source=explicit）。

---

## 4. 核心流程

### 4.1 apply_warming_up 引擎强制流程

```
case_runner.run_case(parsed_steps)
  │
  ├─ [1] 调用 apply_warming_up(parsed_steps)
  │   ├─ 遍历 steps，找 warming_up==true 且 op_id==tg_start_traffic_stream
  │   ├─ 对每个 warming_up step：
  │   │   ├─ 查找其后是否存在 op_id==tg_stop_traffic_stream 的 step
  │   │   ├─ [a] 无显式 post_op：
  │   │   │   ├─ 构造 post_op step = {step_name: "<trigger>_post_op_auto",
  │   │   │   │     op_id: "tg_stop_traffic_stream",
  │   │   │   │     args: {ports, txport, rxport, name} 从主 op 继承,
  │   │   │   │     _warming_up_meta: {source:"auto", trigger_step}}
  │   │   │   └─ 插入到 warming_up step 之后
  │   │   ├─ [b] 有显式 post_op 但缺 ports/txport/rxport/name 之一：
  │   │   │   ├─ 从主 op args 补全缺失字段
  │   │   │   └─ 标记 _warming_up_meta={source:"merged", trigger_step}
  │   │   └─ [c] 有显式 post_op 且参数完整：
  │   │       └─ 标记 _warming_up_meta={source:"explicit", trigger_step}
  │   └─ 返回补全后 steps
  │
  ├─ [2] 逐 step 执行（复用 ST-EX-04 主循环）
  │   ├─ 正常执行 warming_up step（tg_start_traffic_stream）
  │   └─ 执行 post_op step（tg_stop_traffic_stream）
  │       └─ result.json step 记录 auto_post_op 字段（来自 _warming_up_meta.source）
  │
  └─ [3] 逆序清理（ST-EX-06）
      └─ post_op（tg_stop_traffic_stream）是 manual_required 回滚策略，
         handle_rollback 返回 manual_required 不自动回滚（已在 post_op 停流）
```

### 4.2 双重保障执行时序

```
install.py 规则块（ST-EX-02）         case_runner 引擎（ST-EX-08）
        │                                      │
        ├─ 规则文本提示 LLM 写 warming_up       ├─ apply_warming_up 引擎强制
        │  step 后必须写 post_op                │  （无论 md 是否写，引擎兜底）
        │                                      │
        └─ 规则被绕过/重装丢失？                 └─ 引擎在 case_runner 内，
           ↓                                    重装随 skill 安装，不丢失
        RA-008 风险                             ↓
                                           引擎兜底，post_op 强制执行
```

规则侧约束"人"（LLM/用例作者写 md 时遵守），引擎侧约束"机"（case_runner 执行时强制）。即使规则被绕过（md 漏写 post_op），引擎仍补充执行，实现双重保障。

### 4.3 与 ST-EX-06 逆序清理的关系

post_op（tg_stop_traffic_stream）在 op_mapper ROLLBACK_STRATEGY 中是 `manual_required` 类型（handle_rollback 返回 waived 不自动回滚）。这是因为 post_op 本身就是停流清理动作，已在执行阶段完成清理，逆序清理阶段无需重复。apply_warming_up 补充的 post_op step 与用例显式写的 post_op step 在逆序清理中行为一致。

---

## 5. 异常处理

### 5.1 异常分类与处理矩阵

| 异常场景 | 检测方式 | 处理行为 | step 结果 | 用例结果 |
|---|---|---|---|---|
| warming_up step 主 op 执行失败 | envelope.status != success | 不执行 post_op（主 op 未起流，无需停流）；记 warming_up_failed | FAIL | FAIL |
| post_op 执行失败 | envelope.status != success | 记 post_op_failed + auto_post_op=true；触发 TG 残留流告警 | FAIL | FAIL |
| 主 op args 缺 ports/txport/rxport/name | apply_warming_up 参数继承时检测 | 缺失字段记 warning，post_op 用能提取到的字段；缺 name 时 post_op 无法定位流，记 error | ERROR | ERROR |
| post_op 超时 | execute_op TimeoutExpired | envelope error_type=EXEC_FAILED；记 post_op_timeout | FAIL | FAIL |
| post_op 流不存在（tg_stop 返回 stream not found） | ST-EX-11 幂等容错 | 视为期望状态，step 标记 idempotent_skip | PASS | 不降级 |
| post_op STATE_INVALID | op_mapper 自动重连 1 次 | 重连后重试；仍失败记 STATE_INVALID | FAIL | FAIL |

### 5.2 主 op 失败时不执行 post_op 的理由

warming_up step 主 op（tg_start_traffic_stream）若失败，说明流量未发起，不存在残留流量需停。此时执行 post_op 是无效操作（且可能因流不存在报错）。引擎在主 op 失败时跳过 post_op，并在 result.json 记录 `warming_up_failed: true, post_op_skipped: true`。

### 5.3 post_op 失败的严重性

post_op 失败意味着 TG 残留流量未停，会干扰后续用例。step 标记 FAIL，用例结果 FAIL，report.md（ST-EX-11）在诊断中高亮 `post_op_failed` 并提示"需人工检查 TG 残留流"。

---

## 6. 测试设计

### 6.1 测试分层

| 层级 | 范围 | 方式 | 覆盖场景 |
|---|---|---|---|
| 单元测试 | apply_warming_up 纯函数 | Python assert（不引入 pytest，用 case_runner 自带 test 子命令或脚本） | §6.2 |
| dry-run 集成 | 24 用例 warming_up 合规 | case_runner dry-run 扫描 | SM-EX-07 |
| 重装一致性 | 引擎兜底不丢失 | install 循环 | RA-008 |

### 6.2 单元测试用例

| 用例 ID | 输入 | 预期 | 覆盖 |
|---|---|---|---|
| UT-08-01 | steps=[{warming_up:true, op:tg_start, args:{ports,txport,rxport,name}}]（无 post_op） | 返回 2 个 step，第 2 个是 auto post_op，_warming_up_meta.source=auto | 引擎自动补充 |
| UT-08-02 | steps=[{warming_up:true, op:tg_start, args:全}, {op:tg_stop, args:{name}}]（缺 ports/txport/rxport） | 第 2 step 补全 ports/txport/rxport，source=merged | 参数补全 |
| UT-08-03 | steps=[{warming_up:true, op:tg_start, args:全}, {op:tg_stop, args:全}] | 第 2 step source=explicit，不干预 | 显式不干预 |
| UT-08-04 | steps=[{warming_up:false, op:tg_start, args:全}]（无 warming_up） | 返回原 steps，无 post_op 补充 | 非 warming_up 不干预 |
| UT-08-05 | steps=[{warming_up:true, op:tg_start, args:{txport,rxport}}]（缺 ports/name） | post_op args 缺 name 记 warning，source=auto | 缺失字段告警 |
| UT-08-06 | 多个 warming_up step | 每个 warming_up step 后均有对应 post_op | 多预热流场景 |

### 6.3 dry-run 集成测试（SM-EX-07）

`case_runner.py run --cases-dir <24用例> --dry-run` 扫描全部 24 用例，校验：
- 每个 warming_up:true 的 step 执行后，补全后的 steps 列表含对应 post_op
- 输出 warming_up 合规报告：24/24 step 均有强制 post_op

### 6.4 重装一致性测试（RA-008）

```
install.py install ptm-te --component full
case_runner.py run --case-file <warming_up 用例> --dry-run  # 引擎在
install.py uninstall ptm-te
install.py install ptm-te --component full
case_runner.py run --case-file <warming_up 用例> --dry-run  # 引擎仍在
```

验证 apply_warming_up 随 case-execution skill 安装不丢失（与 ST-EX-04 安装验证联动）。

---

## 7. 实施步骤

| 步骤 | 操作 | 文件 | 验证 |
|---|---|---|---|
| 1 | 实现 apply_warming_up 函数 | case_runner.py#apply_warming_up | UT-08-01..06 |
| 2 | 在 run_case 主循环解析 case_steps 后调用 apply_warming_up | case_runner.py#run_case | dry-run 单用例 |
| 3 | step 执行结果记录 auto_post_op 字段（从 _warming_up_meta.source 映射） | case_runner.py#run_case | result.json 含 auto_post_op |
| 4 | 主 op 失败时跳过 post_op 逻辑 | case_runner.py#run_case | UT + dry-run |
| 5 | dry-run 扫描 24 用例 warming_up 合规 | - | SM-EX-07 24/24 |
| 6 | 重装一致性循环验证 | - | RA-008 引擎不丢失 |

### 7.1 完成准则

- apply_warming_up 函数实现 + 6 个单元测试通过
- dry-run 24 用例 warming_up 合规 24/24
- 重装循环后引擎仍生效
- result.json 含 auto_post_op 字段
- CP6 实现执行证据闭环（对象清单/契约映射/测试 fixture）

---

## 8. 回滚策略

### 8.1 代码回滚

ST-EX-08 仅新增 apply_warming_up 函数 + run_case 调用点，不修改既有逻辑。回滚 = 移除 apply_warming_up 函数 + 移除 run_case 中的调用。回滚后 case_runner 退化为"不强制 post_op"，仅依赖 install.py 规则（ST-EX-02），与 exec_v4.py 时代行为一致。

### 8.2 ADR-06 双重保障回退

ADR-06 回退/切换条件：若引擎强制导致用例执行异常（如 post_op 误停非预热流），切到方案 A（只靠规则）。操作：移除 run_case 中 apply_warming_up 调用，保留函数定义（注释禁用），规则侧（ST-EX-02）继续约束。

### 8.3 post_op 失败的运行时回滚

post_op 失败（TG 残留流未停）时，引擎不自动重试停流（避免误停其他用例的流）。report.md 提示人工检查。运行时回滚靠人工：`ptm-atomic run tg trex stop-stream --name <流名>` 手动停流。

---

## 9. 平台差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| ptm-team canonical | skill 源 | apply_warming_up 源在 skills/case-execution/scripts/case_runner.py |
| ptm-te workspace | skill 安装目标 | install.py 安装到 .claude/skills/case-execution/scripts/ |
| ptm-atomic CLI | 不改本体（R-C-001） | post_op 经 op_mapper -> ptm-atomic run tg trex stop-stream |
| TG（trex） | HTTP trex-api | post_op 停流经 ptm-atomic 原子操作，框架不直接调 TG REST API（Gotcha #13） |

无 Claude Code / Codex / Qoder 平台差异——apply_warming_up 是 Python 纯函数，不涉及 agent 平台特性。

---

## 10. 数据模型

### 10.1 case_steps step 结构（warming_up 相关字段）

```yaml
- step_name: "STEP-003"
  op_id: "tg_start_traffic_stream"
  args:
    ports: "${ENV.tg.port1},${ENV.tg.port2}"
    txport: "${ENV.tg.port1}"
    rxport: "${ENV.tg.port2}"
    name: "arp-warmup-flow-01"
    # ... 其他流量参数
  warming_up: true        # 标记 ARP 预热流
  known_issue: false      # 可选，ST-EX-10
  retry: false            # 可选，ST-EX-09
```

### 10.2 补全后 step 结构（含 _warming_up_meta）

```python
{
    "step_name": "STEP-003_post_op_auto",
    "op_id": "tg_stop_traffic_stream",
    "args": {
        "ports": "${ENV.tg.port1},${ENV.tg.port2}",   # 从主 op 继承（占位符原值）
        "txport": "${ENV.tg.port1}",
        "rxport": "${ENV.tg.port2}",
        "name": "arp-warmup-flow-01",
    },
    "_warming_up_meta": {
        "source": "auto",
        "trigger_step": "STEP-003",
    },
}
```

### 10.3 result.json step 记录（warming_up 相关）

```json
{
  "step_name": "STEP-003",
  "op_id": "tg_start_traffic_stream",
  "warming_up": true,
  "status": "success"
}
{
  "step_name": "STEP-003_post_op_auto",
  "op_id": "tg_stop_traffic_stream",
  "auto_post_op": true,
  "source": "auto",
  "trigger_step": "STEP-003",
  "status": "success"
}
```

---

## 11. 性能

### 11.1 apply_warming_up 开销

- 纯内存遍历，O(n) n=step 数（典型 5-15 step/用例），开销可忽略（<1ms）
- 不引入 subprocess / 网络

### 11.2 post_op 执行开销

- post_op（tg_stop_traffic_stream）是 ptm-atomic subprocess 调用，与主 op 同量级
- 每个 warming_up step 增加一次 post_op 调用，24 用例中含 warming_up 的用例增量可控
- 与 exec_v4.py 时代一致（exec_v4 也执行 post_op），无性能回退

### 11.3 性能指标

HLD §13.1 目标 24 用例 < 5 分钟（含 op 间 2s 间隔）。apply_warming_up 不增加显著开销，post_op 已计入用例执行时间。该指标需 runtime 授权后验证（CP7 static-only 不可直接验证）。

---

## 12. 安全

### 12.1 授权边界

- apply_warming_up 是纯解析函数，不触网、不执行 op
- post_op 执行复用 ST-EX-04/05 的授权门：dry-run 默认不执行，--execute --authorized 才实际停流
- post_op 经 ptm-atomic 原子操作，不绕过授权

### 12.2 安全约束遵循

| 约束 | 遵循 |
|---|---|
| NO_CREDENTIAL_READ | 是，apply_warming_up 不读凭据 |
| NO_PRODUCTION_WRITE | 是，post_op 经授权门 + ptm-atomic |
| NO_EXTERNAL_PUBLISH | 是，结果写本地 result.json |
| R-C-001 不改 ptm-atomic | 是，post_op 经 op_mapper -> ptm-atomic |
| Gotcha #13 不直接调 TG REST API | 是，停流经 ptm-atomic run tg trex stop-stream |

---

## 13. 依赖

### 13.1 Story 依赖

| 依赖 Story | 类型 | 依赖内容 | 门控 |
|---|---|---|---|
| ST-EX-04 | hard | case_runner.py 主框架 + run_case 主循环 + case_steps 解析 | dev_gate=cp5-approved |
| ST-EX-03 | soft | op_mapper tg_start/stop_traffic_stream 映射（既有，ST-EX-03 扩展 env_topology 不影响） | lld_gate=full-lld |
| ST-EX-02 | soft | install.py ARP 预热规则块（双重保障的规则侧） | lld_gate=technical-note |

### 13.2 模块依赖

| 模块 | 依赖方向 | 契约 |
|---|---|---|
| op_mapper.execute_op | case_runner -> op_mapper（import，ADR-01） | tg_stop_traffic_stream 映射 + envelope 结构 |
| op_mapper.ROLLBACK_STRATEGY | case_runner 读 | tg_stop_traffic_stream = manual_required（逆序清理不重复） |

### 13.3 假设

- DA-002：24 用例 md 的 case_steps 格式与 parse_steps 兼容（warming_up 字段可解析）
- op_mapper OP_ID_TO_SUBCOMMAND 已含 tg_start_traffic_stream / tg_stop_traffic_stream（既有，已确认）

---

## 14. 开放项

| 问题 ID | 问题 | 状态 | 影响 | 处理 |
|---|---|---|---|---|
| LCQ-ST-EX-08-01 | 多个 warming_up step 共享同一流 name 时，post_op 停流是否相互干扰？ | OPEN（已写 QUESTION-LEDGER） | 若两个 warming_up step 用同 name，第二个 post_op 停流会把第一个的流也停掉 | 待 host-orchestrator 批量问用户；推荐方案：用例 md 规范要求 warming_up 流 name 唯一（ST-EX-13 整改时校验） |
| O-03 | 24 用例 warming_up 标注完整性 | OPEN（ST-EX-13 整改时同步检查） | 若有用例漏标 warming_up，引擎不补充 post_op | ST-EX-13 dry-run 校验 24/24 |

### 14.1 clarification 记录

LCQ-ST-EX-08-01 已写入 `process/state/QUESTION-LEDGER.ndjson`（id=LCQ-ST-EX-08-01, story_id=ST-EX-08, owner_agent=meta-dev, blocks_lld=false）。该问题不阻塞 LLD 定稿（推荐方案在 §14 已给出，ST-EX-13 用例整改时校验 name 唯一即可），待 host-orchestrator 批量问用户确认。
