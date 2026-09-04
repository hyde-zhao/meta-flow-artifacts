---
cr_id: "CR-033"
artifact_type: "feature-design"
feature_id: "FE-EX-02"
feature_name: "case-execution 执行引擎"
version: "1.0"
created_at: "2026-07-28T16:00:00+08:00"
author: "meta-se"
status: "draft"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_blueprint: "docs/design/BLUEPRINT-PTM-TE-EXEC.md"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_dev_plan: "process/DEVELOPMENT-PLAN.yaml"
covered_stories:
  - "ST-EX-04"
  - "ST-EX-05"
  - "ST-EX-06"
  - "ST-EX-07"
  - "ST-EX-08"
  - "ST-EX-09"
  - "ST-EX-10"
  - "ST-EX-11"
  - "ST-EX-12"
  - "ST-EX-14"
  - "ST-EX-15"
  - "ST-EX-16"
  - "ST-EX-17"
validation_mode: "static-only + dry-run-only + review-only"
---

# FE-EX-02 case-execution 执行引擎 Feature 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CP3 approved / CP4 PASS 后生成：case_runner.py 函数签名/类结构/异常处理；frontmatter 16 列解析；case_steps YAML 解析；extract_payload 字段提取表；四态分级判定矩阵；幂等容错规则；report.md 模板；环境解析层（env_topology 契约 + ${ENV.*} 解析 + DUT 接口预配置）；case-execution 安装验证；跨模块契约（case_runner <-> op_mapper import）；覆盖 13 个 Story 设计要点；ADR-01/02/04/05/06/07/09 追溯 |

## 1. Feature 能力边界

### 1.1 能力清单

| 能力 | 说明 | 承载 Story | HLD 章节 |
|---|---|---|---|
| 三入口用例发现 | `--case-file` 单用例 / `--cases-dir` 目录 glob / `--tag` 精确过滤 / `--keyword` 模糊匹配 | ST-EX-04, ST-EX-16 | §12.1 [3] |
| devices.yaml 取址 | 读 `firewall.host`(DUT_URL) + `tg.api_server`(TREX_API_URL fallback) | ST-EX-04 | §12.1 [2], ADR-05 |
| case_steps YAML 解析 | frontmatter 16 列 + case_steps YAML 结构化解析 | ST-EX-04, ST-EX-15 | §12.1 [5], Gotcha #7 |
| dry-run 默认门 | 默认 `--dry-run`，不执行写操作；mutation op 只构建命令并打印 | ST-EX-05 | §12.1 [4], §13.3, ADR-02 |
| --execute 授权门 | `--execute --authorized` 触发真实写操作 + runtime_authorization 审计 | ST-EX-05 | §13.4, ADR-02 |
| 逆序清理 mutation ops | 用例结束后按 config 顺序逆序执行 inverse_op | ST-EX-06 | §12.1 [5], ADR-03 |
| fw_logout 会话清理 | op_mapper fw_logout 映射 + 降级 session 文件清理 | ST-EX-07 | §12.1 [6], ADR-04 |
| ARP 预热引擎强制 | `warming_up:true` 强制 post_op（tg_stop_traffic_stream） | ST-EX-08 | §12.1 [5], ADR-06 |
| retry 轮询 | `retry` 字段解析 + 轮询执行 | ST-EX-09 | §12.1 [5] |
| known_issue 四态分级 | PASS/FAIL/KNOWN_FAIL/ERROR 四态判定 | ST-EX-10 | §12.1 [5], 场景 4 |
| 失败诊断 + 结构化报告 | 失败字段提取 + report.md 四态统计 + 幂等容错记录 | ST-EX-11 | §12.1 [7] |
| extract_payload 统一解析 | 按 op_id 从 envelope.data 提取字段 | ST-EX-12, ST-EX-14 | §10, AGA-04=A |
| verify_loss 消费侧 | tg_verify_traffic_loss 提取 tx/rx/loss_ratio | ST-EX-14 | §10 |
| 用例结构化约定 | 目录三级结构 + 命名正则 + frontmatter 16 列 | ST-EX-15 | §13.5, ADR-07 |
| 标签/关键字执行 | `--tag` 精确 + `--keyword` 模糊 | ST-EX-16 | §12.1 [3] |
| 环境文件驱动 | load_env_file + env_topology 契约 + ${ENV.*} 解析集成 + DUT 接口预配置 | ST-EX-17, ST-EX-04 | §12.3, ADR-09 |
| case-execution 安装验证 | install.py PTM_TE_SKILLS 含 case-execution + 重装一致性 | ST-EX-04 | §12.4, Gotcha #10 |

### 1.2 非目标（Out of Scope）

- 不引入 pytest/robot/外部 eval 框架（R-C-002，HLD §11）
- 不做 HTML 报告（R-C-004，进 BACKLOG）
- 不改 ptm-atomic CLI 本体（R-C-001）
- 不为 devices.yaml 引入 pydantic（R-C-003）
- 不采集 TG 系统快照（R-C-006）
- 不读取凭据/secret/账户（NO_CREDENTIAL_READ）
- 不自动授权真实设备写操作（NO_PRODUCTION_WRITE，--execute 需人工授权）

### 1.3 相邻对象边界澄清

| 相邻对象 | 职责差异 | 边界 |
|---|---|---|
| policy-route-execution (op_mapper) | 映射+执行层；op_id->CLI 映射/build_command/execute_op/rollback/step-refs/STATE_INVALID 重连/resolve_env_refs/TREX_API_URL 注入 | case_runner 只 import 调用，不修改 op_mapper 映射表（ST-EX-03/07 除外，属 FE-EX-01/02 跨 Feature 协作） |
| device-management | 设备清单管理 + 型号映射查表 | case_runner 只读 devices.yaml，不管理设备块（FE-EX-01 职责） |
| traffic-skill / trex-traffic | TG 流量模板 + topology yaml | case_runner 只读 topology yaml/env-file，不 import 代码（R-C-005） |
| ngfw-install | 设备安装 | 不依赖（R-C-005） |
| install.py (FE-EX-03) | 规则块固化 + skill 安装 | case_runner 不负责安装；ST-EX-04 只改 PTM_TE_SKILLS 列表新增 case-execution |

## 2. 数据归属

| 数据对象 | 归属 | 存储位置 | 写入方 | 读取方（本 Feature） |
|---|---|---|---|---|
| case_runner.py | FE-EX-02 | skills/case-execution/scripts/ | meta-dev (ST-EX-04..12,14..17) | install.py 安装 |
| case-execution SKILL.md | FE-EX-02 | skills/case-execution/ | meta-dev (ST-EX-04,15) | install.py 安装 |
| devices.yaml firewall/tg 块 | FE-EX-01 / 既有 | workspace devices.yaml | 测试平台开发者 | case_runner.resolve_addresses 只读 |
| topology yaml / env-file | 既有 / FE-EX-02 加载 | workspace configs/*.yml / --env-file | 既有 | case_runner.load_env_file 只读（ADR-08/09） |
| case_steps YAML | FE-EX-02 解析 / FE-EX-03 整改 | workspace cases/三级/四级/五级/*.md | 测试执行工程师（ST-EX-13 整改） | case_runner.parse_case_file 只读 |
| frontmatter 16 列 | FE-EX-02/03 | workspace cases/*.md | 测试执行工程师 | case_runner.parse_frontmatter 只读（忽略测试步骤/预期结果列，AGA-03=C） |
| result.json | FE-EX-02 | workspace runs/<run-id>/ | case_runner | 测试执行工程师 / 测试经理 |
| report.md | FE-EX-02 | workspace runs/<run-id>/ | case_runner.generate_report | 测试执行工程师 / 测试经理 |
| step-refs/*.json | FE-EX-02 写 / op_mapper 既有 | workspace runs/<run-id>/step-refs/ | op_mapper._write_step_ref | case_runner.run_cleanup 读取（逆序清理） |
| op_mapper 映射表 | FE-EX-01/02 协作 | skills/policy-route-execution/scripts/op_mapper.py | ST-EX-03(resolve_env_refs) / ST-EX-07(fw_logout) | case_runner import 调用 |

## 3. 跨模块契约

### 3.1 case_runner -> op_mapper（import，ADR-01=A）

| 字段 | 值 |
|---|---|
| 调用方向 | case_runner.py 单向 import op_mapper 模块（Layer 4 -> Layer 3） |
| 调用时机 | 逐 step 执行时；预登录；用例后清理；fw_logout 登出；DUT 接口预配置 |
| 触发方式 | Python 函数调用（进程内，无 subprocess 开销） |
| import 路径 | case_runner.py 启动时 `sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../policy-route-execution/scripts'))`（Gotcha #2） |
| 输入契约 | `execute_op(op_id, args, base_url, session_file, *, dry_run, authorized, env_topology, step_id, step_refs_dir, ...)` |
| 输出契约 | envelope dict: `{op_id, step_name, status, data, error_type, diag_snapshot_ref, [runtime_authorization]}` |
| 后续衔接 | case_runner 读 envelope.status/error_type 判定四态 + 记录 mutation op 供逆序清理 |
| 降级策略 | op_mapper OpNotFoundError -> envelope error_type=OP_NOT_FOUND；ValidationError -> PARAM_INVALID；fw_logout 未暴露 -> case_runner.cleanup_session 降级清理 session 文件（ADR-04） |
| 调用方需同步修改范围 | op_mapper 新增 resolve_env_refs + _build_exec_env 签名扩展（ST-EX-03，FE-EX-01 LLD 承载）；op_mapper 新增 fw_logout 映射（ST-EX-07，本 Feature） |

### 3.2 case_runner -> devices.yaml（只读）

| 字段 | 值 |
|---|---|
| 调用方向 | case_runner.resolve_addresses 读 devices.yaml |
| 调用时机 | 启动校验后、预登录前（§12.1 [2]） |
| 输入契约 | devices.yaml 路径（--devices-yaml 参数） |
| 输出契约 | `{dut_url: firewall.host, tg_api_server: tg.api_server}` |
| 前置校验 | DA-003：devices.yaml 含 firewall.host 和 tg.api_server，缺失时 fail fast |
| 降级策略 | tg_api_server 可被 env_topology ${ENV.tg.url} 覆盖（ADR-05 环境文件优先） |

### 3.3 case_runner -> topology yaml / env-file（只读，ADR-08/09）

| 字段 | 值 |
|---|---|
| 调用方向 | case_runner.load_env_file 读 --env-file（或 --topology-yaml 兼容） |
| 调用时机 | 启动校验后、预登录前（§12.1 [2]） |
| 输入契约 | env-file 路径；结构 port_mapping/nodes/links |
| 输出契约 | env_topology dict（见 §7.1） |
| 前置校验 | DA-006：文件存在 + port_mapping/nodes/links 字段完整；DA-007：port_mapping 覆盖用例引用全部逻辑端口 |
| 降级策略 | env-file 缺失时 TREX_API_URL fallback devices.yaml（ADR-05）；端口/IP 缺失报错终止（RA-014） |

### 3.4 case_runner -> install.py（PTM_TE_SKILLS，ST-EX-04）

| 字段 | 值 |
|---|---|
| 调用方向 | ST-EX-04 修改 install.py PTM_TE_SKILLS 列表新增 "case-execution" |
| 调用时机 | install.py install ptm-te --component full 时 |
| 输入契约 | PTM_TE_SKILLS = [..., "case-execution"] |
| 输出契约 | workspace .claude/skills/case-execution/ 安装 |
| 验证 | 重装 install -> uninstall -> install 循环验证 case-execution skill 不丢失（Gotcha #10） |

## 4. 接口设计（case_runner.py 函数签名）

> 函数签名与 DEVELOPMENT-PLAN.yaml file_ownership 一一对应。详细实现由各 Story LLD/技术说明承载，本节定契约。

### 4.1 CLI 入口与用例发现（ST-EX-04, ST-EX-16）

```python
def main(argv: Optional[List[str]] = None) -> int:
    """argparse CLI 入口。

    子命令: run
    三入口（互斥组）: --case-file <path> | --cases-dir <dir>
    过滤: --tag <tags>（精确，逗号分隔）| --keyword <kw>（模糊）
    设备: --devices-yaml <path>（必填）
    拓扑: --topology-yaml <path>（TG step 必填，ADR-08）
    环境: --env-file <path>（含 ${ENV.*} 必填，ADR-09）
    模式: --dry-run（默认）| --execute --authorized
    输出: --output-dir <dir>（默认 runs/<run-id>/）
    """

def discover_cases(
    case_file: Optional[str],
    cases_dir: Optional[str],
    tag: Optional[str],
    keyword: Optional[str],
) -> List[CaseRecord]:
    """用例发现：--case-file 单用例 / --cases-dir glob *.md；
    --tag 按 frontmatter tags 列精确过滤；--keyword 模糊匹配。"""

def parse_case_file(path: str) -> CaseRecord:
    """解析单个用例 md：frontmatter 16 列 + case_steps YAML。"""

def parse_frontmatter(text: str) -> Frontmatter:
    """frontmatter 16 列解析（ST-EX-15）。
    必填 8 列 + 可选 8 列（DQ-05）；忽略'测试步骤'/'预期结果'列（AGA-03=C，Gotcha #7）。"""

def filter_by_tag(cases: List[CaseRecord], tag: str) -> List[CaseRecord]:
    """--tag 精确过滤：frontmatter tags 列含指定标签（逗号分隔多标签）。"""

def filter_by_keyword(cases: List[CaseRecord], keyword: str) -> List[CaseRecord]:
    """--keyword 模糊匹配：frontmatter 标题/描述含关键字。"""
```

### 4.2 取址与环境加载（ST-EX-04, ST-EX-17）

```python
def resolve_addresses(devices_yaml: str) -> dict:
    """从 devices.yaml 读 firewall.host -> DUT_URL, tg.api_server -> TREX_API_URL fallback。
    前置校验 DA-003：字段缺失 fail fast。"""

def load_env_file(env_file: Optional[str]) -> Optional[EnvTopology]:
    """加载 --env-file -> env_topology（ST-EX-04 加载 / ST-EX-17 校验）。
    env_file=None 时返回 None（向后兼容，Gotcha #12）。
    YAML 1.1 整数陷阱规避：port_mapping 端口名必须加引号（Gotcha #11）。"""

def build_env_topology(env_file: str) -> EnvTopology:
    """校验 env-file 完整性（ST-EX-17）：
    - port_mapping/nodes/links 字段存在（DA-006）
    - port_mapping 覆盖用例引用全部逻辑端口（DA-007）
    - 端口名为字符串类型（Gotcha #11）
    返回 EnvTopology（port_mapping/nodes/links）。"""

def preconfigure_dut_interfaces(
    env_topology: EnvTopology,
    base_url: str, session_file: str,
    *, dry_run: bool, authorized: bool,
) -> List[MutationRecord]:
    """DUT 接口自动预配置（ST-EX-17，R-F-028）：
    --execute 模式按 nodes.dut1.interfaces 调用 op_mapper fw_update_interface。
    返回预配置记录供用例后逆序还原。
    清理顺序：先 ST-EX-06 用例 mutation ops 逆序清理，再本函数预配置接口逆序还原（HLD §12.3）。"""
```

### 4.3 执行编排（ST-EX-04, ST-EX-05, ST-EX-08, ST-EX-09, ST-EX-10, ST-EX-12, ST-EX-14）

```python
def execute_steps(
    steps: List[Step], *,
    base_url: str, session_file: str,
    dry_run: bool, authorized: bool,
    env_topology: Optional[EnvTopology],
    step_refs_dir: str,
    timeout_tg: int = 90, timeout_dut: int = 45,  # Gotcha #9
) -> List[StepResult]:
    """逐 step 执行（ST-EX-05）：
    1. resolve_env_refs（op_mapper，${ENV.*} 插值，ADR-09）
    2. resolve_step_refs（op_mapper，${STEP-N.id} 插值）
    3. validate_args（op_mapper）
    4. build_command（op_mapper）
    5. dry-run 默认门 / --execute 授权门（authorize）
    6. op_mapper.execute_op(env_topology=env_topology)
    7. STATE_INVALID 重连（op_mapper，最多 1 次）
    8. ConnectTimeout 重试（TG 3 次 15/20/25s / DUT 1 次 30s）
    9. 幂等容错判定（§6.2）
    10. known_issue 标记 -> KNOWN_FAIL（classify_result）
    11. retry 字段 -> 轮询（apply_retry）
    12. warming_up:true -> 强制 post_op（apply_warming_up）
    fw_login_web_management step 标记 skipped（共享 session，Gotcha #8）。"""

def authorize(dry_run: bool, authorized: bool) -> Optional[dict]:
    """--execute 授权门（ST-EX-05，ADR-02）：
    dry_run=False 且 authorized=False -> 拒绝（error_type=EXEC_FAILED）；
    dry_run=False 且 authorized=True -> 返回 runtime_authorization 审计字段。"""

def apply_warming_up(step: Step, result: StepResult, ...) -> Optional[StepResult]:
    """ARP 预热引擎强制（ST-EX-08，ADR-06）：
    step.warming_up=true 时，主 op(tg_start_traffic_stream) 执行后强制 post_op(tg_stop_traffic_stream)；
    参数从主 op 继承（ports/txport/rxport/name）；result 标记 auto_post_op=true。"""

def apply_retry(step: Step, result: StepResult, ...) -> StepResult:
    """retry 轮询（ST-EX-09）：step.retry={interval, count, until} 解析 + 轮询执行。"""

def classify_result(steps: List[StepResult]) -> str:
    """四态分级（ST-EX-10）：
    PASS（全 success）/ FAIL（含 fail 非 known_issue）/ KNOWN_FAIL（含 fail 且 known_issue=true）/ ERROR（含 error）。
    优先级：ERROR > KNOWN_FAIL > FAIL > PASS。"""

def extract_payload(op_id: str, envelope: dict) -> dict:
    """统一解析（ST-EX-12/14，AGA-04=A）：按 op_id 从 envelope.data 提取字段。
    字段提取表见 §6.1。"""

def run_cleanup(
    mutation_ops: List[MutationRecord], *,
    base_url: str, session_file: str,
    dry_run: bool, authorized: bool,
    env_topology: Optional[EnvTopology],
) -> List[CleanupRecord]:
    """逆序清理（ST-EX-06）：按 config 顺序逆序执行 inverse_op（op_mapper.handle_rollback）；
    step-refs 读取前序 step 的 id；irreversible 类不回滚。"""
```

### 4.4 会话清理与报告（ST-EX-07, ST-EX-11）

```python
def cleanup_session(
    base_url: str, session_file: str, *,
    dry_run: bool, authorized: bool,
    env_topology: Optional[EnvTopology],
) -> dict:
    """fw_logout 会话清理（ST-EX-07，ADR-04）：
    1. op_mapper.execute_op(fw_logout) 登出
    2. ptm-atomic 未暴露 fw_logout -> 降级 os.remove(session_file)
    3. result.json 记录 logout 状态（success/fallback_session_cleanup）"""

def diagnose_failure(envelope: dict) -> Diagnostics:
    """失败诊断（ST-EX-11）：提取 error_type/error_code/reason/details/command。"""

def generate_report(results: List[CaseResult], output_dir: str) -> str:
    """生成 report.md（ST-EX-11）：四态统计表 + 失败 step 诊断 + 幂等容错记录。模板见 §6.3。"""
```

## 5. frontmatter 16 列解析（ST-EX-15）

### 5.1 16 列定义（DQ-05：8 必填 + 8 可选）

| 序 | 列名 | 必填 | 说明 | case_runner 处理 |
|---|---|---|---|---|
| 1 | 用例编号 | 是 | 唯一标识（如 PC-COMB-M4-01-01），ADR-07 正则 `^PC-[A-Z0-9]+-\d+-\d+-\d+` | 用例 ID |
| 2 | 用例标题 | 是 | 用例名称 | 报告显示 |
| 3 | 用例目的 | 是 | 测试意图 | 报告显示 |
| 4 | 特性 | 是 | 所属特性 | 目录归属 |
| 5 | 优先级 | 是 | P0/P1/P2/P3 | 排序 |
| 6 | 标签 | 是 | tags 列，逗号分隔 | --tag 过滤（ST-EX-16） |
| 7 | 关键字 | 是 | 模糊匹配关键词 | --keyword 过滤（ST-EX-16） |
| 8 | 前置条件 | 是 | 环境前置 | 校验 |
| 9 | 测试步骤 | 否 | 与 case_steps YAML 冗余 | **忽略**（AGA-03=C，Gotcha #7） |
| 10 | 预期结果 | 否 | 与 case_steps expected_result 冗余 | **忽略**（AGA-03=C） |
| 11 | 用例类型 | 否 | 正向/反向 | 报告分类 |
| 12 | 自动化状态 | 否 | auto/manual | 报告分类 |
| 13 | 创建人 | 否 | 审计 | 报告显示 |
| 14 | 创建日期 | 否 | 审计 | 报告显示 |
| 15 | known_issue | 否 | DUT 行为差异标记 | 四态分级（ST-EX-10） |
| 16 | 备注 | 否 | 补充 | 报告显示 |

缺失可选列填 `N/A`（RA-012）。校验脚本检查 16 列存在性，不校验"测试步骤"/"预期结果"内容一致性。

### 5.2 文件命名解析（ADR-07，Gotcha #6）

文件名格式：`<编号>-<名称>.md`，编号含连字符（PC-COMB-M4-01-01），名称也含连字符。解析按 frontmatter 用例编号列为唯一标识；文件名按编号前缀正则 `^(PC-[A-Z0-9]+-\d+-\d+-\d+)` 匹配，剩余为名称。

## 6. case_steps YAML 解析与字段表（ST-EX-04, ST-EX-08, ST-EX-09, ST-EX-10, ST-EX-12, ST-EX-14）

### 6.1 extract_payload 字段提取表（ST-EX-12/14，AGA-04=A）

| op_id | 提取字段 | envelope.data 路径 | 用途 |
|---|---|---|---|
| fw_config_policy_route | policy_route_id | data.policy_route_id | update/delete/reset 引用 |
| fw_config_object | object_id | data.object_id | 引用 |
| fw_config_interface | interface_id | data.interface_id | delete 引用 |
| tg_config_interface | port_status | data.interfaces[].status | 预配置校验 |
| tg_apply_traffic_template | template_id | data.template_id | start/delete 引用 |
| tg_start_traffic_stream | stream_handle | data.handle | stop 引用 |
| tg_verify_traffic_loss | tx/rx/loss_ratio | data.tx_count/data.rx_count/data.loss_ratio | verify_loss 消费侧（ST-EX-14） |
| fw_verify_policy_route | hitcount | data.hitcount | 断言 |
| 其他 | 无 | - | envelope 原样透传 |

### 6.2 幂等容错规则（ST-EX-11）

| error_type / 场景 | 容错判定 | 结果 |
|---|---|---|
| 对象已存在（fw_config_object/fw_config_policy_route 返回 already_exists） | 视为期望状态 | success(idempotent=true) |
| 流不存在（tg_stop_traffic_stream 返回 not_found） | 视为期望状态 | success(idempotent=true) |
| 被引用阻止删除（fw_delete_object 返回 eBeingReferenced + known_issue=true） | KNOWN_FAIL | KNOWN_FAIL |
| STATE_INVALID | op_mapper 自动重连 1 次后重试 | success/error |
| ConnectTimeout | TG 重试 3 次 / DUT 重试 1 次 | success/error |

### 6.3 report.md 模板（ST-EX-11）

```markdown
# 测试执行报告

**运行 ID**: <run-id>
**执行时间**: <timestamp>
**模式**: dry-run | --execute（授权: who/scope/authorized_at）
**设备**: DUT_URL=<dut_url>, TREX_API_URL=<tg_url>

## 四态统计

| 状态 | 数量 | 用例编号 |
|---|---|---|
| PASS | N | PC-... |
| FAIL | N | PC-... |
| KNOWN_FAIL | N | PC-... |
| ERROR | N | PC-... |

## 失败 step 诊断

### <用例编号> - <step_id>
- op_id: <op_id>
- error_type: <error_type>
- error_code: <error_code>
- reason: <reason>
- details: <details>
- command: <command>
- known_issue: true/false

## 幂等容错记录

| step_id | op_id | 容错场景 | 结果 |
|---|---|---|---|
| STEP-003 | fw_config_object | 对象已存在 | success(idempotent) |

## 清理记录

| 用例 | 清理 op | 状态 |
|---|---|---|
| PC-... | fw_delete_policy_route | success |
```

### 6.4 case_steps YAML 字段格式

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 登录
    op_id: fw_login_web_management
    args: {username: admin, password_env: FW_WEB_PASSWORD}
    target: dut                      # dut | tg，决定 timeout（Gotcha #9）
    warming_up: false                # ST-EX-08，true 时强制 post_op
    post_op:                         # 可选，warming_up=true 时引擎自动补充
      op_id: tg_stop_traffic_stream
      args_from: main                # 从主 op 继承 ports/txport/rxport/name
    retry:                           # ST-EX-09，可选
      interval: 5
      count: 3
      until: "status==success"
    known_issue: false               # ST-EX-10，true 时 fail -> KNOWN_FAIL
    expected_result: success
```

## 7. 环境解析层设计（ST-EX-17，ADR-09，CP3 评审范围扩展）

### 7.1 env_topology 契约（HLD §12.3）

```yaml
# env-file schema（case_runner.load_env_file 只读，CR-033 暂用 topology yaml 载体，ADR-08）
port_mapping:
  port1: {tg: "1/1/1", dut: "eth0"}     # 逻辑端口 -> 物理映射
  port2: {tg: "1/1/2", dut: "eth1"}
nodes:
  tg1:
    trex_api_url: "http://10.113.52.253:8450"   # TREX_API_URL 来源（ADR-05 环境文件优先）
    interfaces:
      port1: {ip: "192.168.101.1", gw: "192.168.101.2"}
      port2: {ip: "192.168.102.1", gw: "192.168.102.2"}
  dut1:
    host: "https://10.113.55.51"                # DUT_URL
    next_hop: "192.168.102.1"                    # policy-route next_hop_ip
    interfaces:
      port1: {ip: "192.168.101.2"}
      port2: {ip: "192.168.102.2"}
links:
  - {from: "tg1.port1", to: "dut1.port1", name: "link1"}
  - {from: "tg1.port2", to: "dut1.port2", name: "link2"}
```

### 7.2 ${ENV.*} 占位符解析规则（9 类，HLD §12.3，SM-EX-12）

| 占位符 | 解析目标 | 说明 |
|---|---|---|
| `${ENV.tg.port1}` | port_mapping.port1.tg | TG 物理端口名 |
| `${ENV.tg.port1.ip}` | nodes.tg1.interfaces.port1.ip | TG 端口 IP |
| `${ENV.tg.port1.gw}` | nodes.tg1.interfaces.port1.gw | TG 端口网关 |
| `${ENV.dut.port1}` | port_mapping.port1.dut | DUT 物理端口名 |
| `${ENV.dut.port1.ip}` | nodes.dut1.interfaces.port1.ip | DUT 端口 IP |
| `${ENV.tg.url}` | nodes.tg1.trex_api_url | TG api_server URL（TREX_API_URL 来源，ADR-05） |
| `${ENV.dut.url}` | nodes.dut1.host | DUT URL |
| `${ENV.dut.next_hop}` | nodes.dut1.next_hop | DUT 下一跳 IP（policy-route next_hop_ip） |
| `${ENV.tg.ports[port1,port2]}` | [port_mapping.port1.tg, port_mapping.port2.tg] | 聚合数组 |

**执行顺序**（op_mapper.execute_op 内）：`resolve_env_refs -> resolve_step_refs -> validate_args -> build_command`（ADR-09）。

**向后兼容**：args 未含 ${ENV.*} 字面值原样透传；env_topology=None 时原样透传（Gotcha #12）。dry-run 模式下 resolve_env_refs 仍执行（验证解析正确性，HLD §13.3）。

**失败行为**：resolve_env_refs 解析失败（占位符无对应 env_topology 键）-> VALIDATION_FAILED envelope，step 标记 error（RA-013）。

### 7.3 DUT 接口自动预配置流程（ST-EX-17，R-F-028）

```
--execute 模式：
  1. build_env_topology 校验 env-file（DA-006/007）
  2. preconfigure_dut_interfaces：按 nodes.dut1.interfaces 调 op_mapper fw_update_interface
  3. 记录预配置 MutationRecord 供清理
  4. 执行用例 case_steps
  5. 用例后清理（顺序严格）：
     a. ST-EX-06 run_cleanup：用例 case_steps mutation ops 逆序清理
     b. ST-EX-17 preconfigure_dut_interfaces 逆序还原（预配置接口后还原）
     确保用例写的 op 先回滚，框架预配置的接口后还原（HLD §12.3）
```

### 7.4 ptm-atomic 约束（Gotcha #13，ADR-05/09）

TG 操作必须经 `ptm-atomic run tg trex <action>` 原子操作下发。环境文件驱动仅做参数解析（resolve_env_refs）与 TREX_API_URL 注入（op_mapper._build_exec_env 环境变量），由 ptm-atomic 子进程消费。框架禁止直接调 TG REST API（code review 检查无 requests/urllib 直接调 TG 代码）。

## 8. 异常处理

| 异常来源 | 处理 | error_type |
|---|---|---|
| op_mapper.OpNotFoundError | envelope error | OP_NOT_FOUND |
| op_mapper.ValidationError | envelope error | PARAM_INVALID |
| resolve_env_refs 解析失败 | envelope error | VALIDATION_FAILED |
| subprocess.TimeoutExpired | envelope error | EXEC_FAILED |
| STATE_INVALID | op_mapper 自动重连 1 次 | STATE_INVALID -> success/error |
| fw_logout 未暴露 | 降级 os.remove(session_file) | logout=fallback_session_cleanup |
| devices.yaml 字段缺失 | 启动 fail fast | - |
| env-file 字段不完整 | 启动 fail fast（DA-006/007） | - |

case_runner 不向上抛未捕获异常：所有 step 异常转为 envelope error，记入 result.json；启动校验失败 fail fast 退出码非 0。

## 9. 与 HLD 一致性 + ADR 追溯

| ADR | 决策 | 本 Feature 落点 | HLD 章节 |
|---|---|---|---|
| ADR-01 | case_runner 直接 import op_mapper（A） | §3.1 跨模块契约；§4 函数签名 import 调用 | §9 架构图 / §10 模块表 |
| ADR-02 | dry-run 默认门 + --execute 授权 | §4.3 execute_steps/authorize；§1.1 能力 | §12.1 [4] / §13.4 |
| ADR-03 | exec_v4.py 废弃标记 | ST-EX-13 承载（FE-EX-03）；本 Feature 不删除 | §14 RA-009 |
| ADR-04 | fw_logout 映射 + 降级 | §4.4 cleanup_session；§8 异常处理 | §12.1 [6] / Gotcha #4 |
| ADR-05 | TREX_API_URL _build_exec_env 注入（环境文件优先） | §3.2 resolve_addresses fallback；§7.2 ${ENV.tg.url} 优先 | §12.2 / §12.3 / Gotcha #3 |
| ADR-06 | ARP 预热双重保障 | §4.3 apply_warming_up（引擎强制）+ FE-EX-03 规则 | §12.1 [5] / Gotcha #8 |
| ADR-07 | 用例编号正则匹配 | §5.2 文件命名解析 | Gotcha #6 |
| ADR-09 | 环境文件驱动 | §7 环境解析层（全文） | §12.3 / Gotcha #11/12/13 |

**量化指标对齐**（HLD §成功标准）：SM-EX-02 三入口（§4.1）/ SM-EX-03 零硬编码（§3.2 取址）/ SM-EX-04 extract_payload 1 函数（§4.3/§6.1）/ SM-EX-08 四态（§4.3 classify_result）/ SM-EX-10 fw_logout op（§4.4）/ SM-EX-11 op_id 22（ST-EX-07 EXPECTED_OP_COUNT）/ SM-EX-12 ${ENV.*} 9 类（§7.2）。

## 10. Story 设计要点

| Story | lld_policy | 设计要点 | 证据路径 |
|---|---|---|---|
| ST-EX-04 | full-lld | case_runner.py 三入口 + devices.yaml 取址 + case_steps 解析 + load_env_file + case-execution 安装验证（PTM_TE_SKILLS） | process/stories/STORY-EX-04-LLD.md |
| ST-EX-05 | full-lld | dry-run 默认门 + --execute 授权门 + runtime_authorization 审计 | process/stories/STORY-EX-05-LLD.md |
| ST-EX-06 | full-lld | 逆序清理 mutation ops + step-refs 读取 + rollback | process/stories/STORY-EX-06-LLD.md |
| ST-EX-07 | full-lld | op_mapper fw_logout 映射 + EXPECTED_OP_COUNT 21->22 + 降级 session 清理 | process/stories/STORY-EX-07-LLD.md |
| ST-EX-08 | full-lld | warming_up/post_op 引擎强制（ADR-06 双重保障引擎侧） | process/stories/STORY-EX-08-LLD.md |
| ST-EX-09 | technical-note | retry 字段解析 + 轮询执行 | process/stories/STORY-EX-09.md |
| ST-EX-10 | technical-note | known_issue 标记 + 四态判定矩阵 | process/stories/STORY-EX-10.md |
| ST-EX-11 | full-lld | 失败诊断字段提取 + report.md 模板 + 幂等容错记录 | process/stories/STORY-EX-11-LLD.md |
| ST-EX-12 | technical-note | extract_payload(op_id, envelope) 字段提取表 | process/stories/STORY-EX-12.md |
| ST-EX-14 | technical-note | tg_verify_traffic_loss 提取 tx/rx/loss_ratio | process/stories/STORY-EX-14.md |
| ST-EX-15 | technical-note | 目录三级结构 + 命名正则 + frontmatter 16 列约定 | process/stories/STORY-EX-15.md |
| ST-EX-16 | technical-note | --tag 精确过滤 + --keyword 模糊匹配 | process/stories/STORY-EX-16.md |
| ST-EX-17 | full-lld | env_topology 契约 + ${ENV.*} 解析集成 + DUT 接口预配置 + 用例后清理 + TREX_API_URL 环境文件优先 | process/stories/STORY-EX-17-LLD.md |

**Story 数一致性**：本 Feature 覆盖 13 个 Story（ST-EX-04,05,06,07,08,09,10,11,12,14,15,16,17），与 FEATURE-DESIGN-MATRIX FE-EX-02 关联 Story 列、BLUEPRINT §1.2 Epic 拆解、DEVELOPMENT-PLAN file_ownership 一致。

## 11. 前置校验与失败路径

| 阶段 | 前置校验 | 失败行为 |
|---|---|---|
| 启动 | which ptm-atomic（DA-001） | fail fast 退出 |
| 启动 | devices.yaml 含 firewall.host + tg.api_server（DA-003） | fail fast |
| 启动 | topology yaml 存在 + TG 接口映射（DA-006，TG step 时） | fail fast |
| 启动 | env-file port_mapping/nodes/links 完整（DA-006/007，含 ${ENV.*} 时） | fail fast |
| 预登录 | fw_login_web_management 建立 SHARED_SESSION | ConnectTimeout 重试 3 次 |
| 逐 step | dry-run 默认门 / --execute 授权 | 未授权 -> EXEC_FAILED |
| 逐 step | resolve_env_refs 解析 | 失败 -> VALIDATION_FAILED |
| 逐 step | STATE_INVALID | op_mapper 重连 1 次 |
| 逐 step | ConnectTimeout | TG 3 次 / DUT 1 次 |
| 清理 | inverse_op 执行 | 失败记录 cleanup error |
| 登出 | fw_logout | 未暴露 -> 降级 session 文件清理 |

## 12. Gotchas

### Gotcha FE02-1: op_mapper EXPECTED_OP_COUNT 必须同步更新（ST-EX-07）

新增 fw_logout 后 `EXPECTED_OP_COUNT` 必须从 21 改为 22，否则 `validate_mapping_consistency()` 报 "应覆盖 21 个 op_id，实际 22 个"。auth 族子命令校验新增 logout action。（HLD Gotcha #1）

### Gotcha FE02-2: case_runner import op_mapper 路径（ST-EX-04）

case_runner.py 在 `skills/case-execution/scripts/`，op_mapper.py 在 `skills/policy-route-execution/scripts/`。直接 import 因 sys.path 不含对方目录失败。启动时 `sys.path.insert(0, '../../policy-route-execution/scripts')` 显式添加。（HLD Gotcha #2）

### Gotcha FE02-3: TREX_API_URL 注入需扩展 _build_exec_env 签名（ST-EX-03 协作）

op_mapper._build_exec_env(base_url) 当前只接收 base_url。ST-EX-03 扩展签名新增 `tg_api_server: str = ""`；case_runner 调用 execute_op 时传入（env_topology ${ENV.tg.url} 优先，devices.yaml fallback）。方案 a 定稿（ADR-05），不采用方案 b（os.environ 直读）。（HLD Gotcha #3）

### Gotcha FE02-4: fw_logout 可能未在 ptm-atomic 安装版暴露（ST-EX-07）

安装前 `ptm-atomic show fw_logout` 验证；未暴露时降级 os.remove(session_file)，result.json 记录 logout=fallback_session_cleanup。（HLD Gotcha #4，ADR-04）

### Gotcha FE02-5: 共享 session 跨用例复用，fw_login step 跳过（ST-EX-04/05）

预登录建立 SHARED_SESSION 后，逐用例遇 fw_login_web_management step 标记 skipped。dry-run 模式不预登录，fw_login step 只构建命令不执行（mutation op dry-run 跳过）。（HLD Gotcha #8）

### Gotcha FE02-6: TG op timeout 需大于 DUT op timeout（ST-EX-04）

tg_config_interface 含 ARP 学习耗时 90s；DUT op 45s。execute_steps 按 step.target 设 timeout。（HLD Gotcha #9）

### Gotcha FE02-7: frontmatter 测试步骤/预期结果列冗余（ST-EX-15）

case_runner 忽略 frontmatter 这两列，只读 case_steps YAML（AGA-03=C）。校验脚本只检查 16 列存在性，不校验内容一致性。（HLD Gotcha #7）

### Gotcha FE02-8: YAML 1.1 整数陷阱--端口名必须加引号（ST-EX-17）

env-file port_mapping 物理端口名（如 "1/1/1"）不加引号会被 YAML 1.1 解析为整数/日期。load_env_file 校验端口名为字符串类型，非字符串报 VALIDATION_FAILED。（HLD Gotcha #11）

### Gotcha FE02-9: 向后兼容--无 ${ENV.*} 字面值原样透传（ST-EX-17）

resolve_env_refs 对未含 ${ENV.*} 的 args 原样透传；env_topology=None 时原样透传。dry-run 模式下 resolve_env_refs 也执行。（HLD Gotcha #12）

### Gotcha FE02-10: ptm-atomic 约束--框架禁止直接调 TG REST API（ST-EX-17）

resolve_env_refs 只解析参数不发起网络请求；TREX_API_URL 经 _build_exec_env 注入子进程环境变量，由 ptm-atomic 子进程消费。code review 检查无 requests/urllib 直接调 TG 代码。（HLD Gotcha #13）

### Gotcha FE02-11: DUT 接口预配置清理顺序（ST-EX-17）

用例后清理必须先 ST-EX-06 用例 mutation ops 逆序清理，再 ST-EX-17 框架预配置接口逆序还原。顺序颠倒会导致用例引用的接口先被还原，清理 op 失败。（HLD §12.3）

## 13. 待确认问题

| 问题 ID | 问题 | 状态 | 决策引用 |
|---|---|---|---|
| O-FE02-01 | fw_logout op 在 ptm-atomic 安装版是否暴露 | OPEN（安装前验证） | ADR-04 / DQ-CP3-02 |
| O-FE02-02 | 24 用例 known_issue 标注完整性（ST-EX-13 协作） | OPEN（整改时同步检查） | DQ-CP3-03 / RA-003 |
| O-FE02-03 | 环境文件 schema 管理 skill/CLI（后续 CR 候选） | OPEN（CR-033 范围外） | ADR-08/09 / O-04 |

## 14. 验证策略（与 TEST-PLAN.md 联动）

本 Feature `validation_mode=static-only + dry-run-only + review-only`（HLD §13.1/§13.3）。CP7 不真实设备写：
- static-only：code review 检查 import 路径/异常处理/无直接 TG REST 调用
- dry-run-only：case_runner --dry-run 批量校验 24 用例解析 + resolve_env_refs + build_command
- review-only：report.md 模板/四态矩阵/extract_payload 字段表人工审查

真实设备 --execute 验证需 runtime_authorization，超出 CR-033 CP7 范围（HLD §13.1 性能指标在 static-only 下不可直接验证）。详见 `TEST-PLAN.md`。
