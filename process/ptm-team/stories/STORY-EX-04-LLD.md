---
cr_id: "CR-033"
story_id: "STORY-EX-04"
story_slug: "case-runner-core"
artifact_type: "lld"
lld_policy_required_level: "full-lld"
design_evidence_type: "full-lld"
feature_design_refs:
  - "docs/features/case-execution/DESIGN.md"
version: "1.0"
created_at: "2026-07-28T16:00:00+08:00"
author: "meta-dev"
status: "draft"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_feature_matrix: "docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md"
source_dev_plan: "process/DEVELOPMENT-PLAN.yaml"
file_ownership:
  - "skills/case-execution/SKILL.md"
  - "skills/case-execution/scripts/case_runner.py#main"
  - "skills/case-execution/scripts/case_runner.py#discover_cases"
  - "skills/case-execution/scripts/case_runner.py#parse_case_file"
  - "skills/case-execution/scripts/case_runner.py#resolve_addresses"
  - "script/ptm_team/install.py#PTM_TE_SKILLS"
depends_on: ["ST-EX-03"]
dependency_type: "hard"
---

# ST-EX-04 case_runner 核心引擎 LLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-dev | ST-EX-04 full-lld 初稿：14 章节全覆盖。三入口 + devices.yaml 取址 + case_steps 解析 + load_env_file 加载 + preconfigure_dut_interfaces 编排调用 + _build_exec_env TREX_API_URL 消费契约 + case-execution 安装验证；与 ST-EX-03/05/06/07/08/17 边界显式划分；LCQ-ST-EX-04-01（--env-file 与 --topology-yaml 关系，非阻塞） |

## 0. 概述

本 LLD 是 ST-EX-04 case_runner 核心引擎的 full-lld 设计证据，属于 FE-EX-02 case-execution 引擎 Feature。ST-EX-04 是 CR-033 ptm-te 执行引擎的核心 Story，承载 case_runner.py 编排层从零新建。

**需求溯源**：ST-EX-04 承载 R-F-004（用例执行入口三入口）、R-F-005（devices.yaml 取址消除硬编码）、R-F-006（dry-run 校验）、R-F-027（--env-file 加载）、R-F-028（DUT 接口预配置编排）等功能需求；详细需求映射见 §1.1 设计产物溯源。

**模块拆分**：case_runner.py 作为编排层，按职责拆分为 main（CLI 入口）/discover_cases（用例发现）/parse_case_file（用例解析）/resolve_addresses（设备取址）/load_env_file（环境加载）等函数锚点（ST-EX-04 拥有）；下游 Story 填充 execute_steps（ST-EX-05）/run_cleanup（ST-EX-06）/cleanup_session（ST-EX-07）/apply_warming_up（ST-EX-08）/generate_report（ST-EX-11）/build_env_topology+preconfigure_dut_interfaces（ST-EX-17）等函数。

**代码结构**：详见 §2 文件影响范围。case_runner.py 与 op_mapper.py 跨 skill import（ADR-01），sys.path 注入见 §5.3；install.py PTM_TE_SKILLS 新增 case-execution 实现安装验证。

**技术细节**：TREX_API_URL 注入消费契约（§3.6，ADR-05 环境文件优先 ${ENV.tg.url}，devices.yaml fallback）、环境文件驱动 load_env_file（§3.5，ADR-09）、清理顺序编排（§4.2，先 ST-EX-06 mutation ops 逆序清理再 ST-EX-17 预配置接口逆序还原）、dry-run 默认门（§4.3，ADR-02）等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试设计的 23 单元测试 + 4 dry-run 校验 + 4 安装验证全通过为准；CP7 验证采用 static-only + dry-run-only（DQ-01 推荐），--execute 集成测试需独立 runtime_authorization。DoD 详见 §6.2/§6.3/§6.4 验证矩阵。

## 1. 工程依据

### 1.1 设计产物溯源

| 依据类型 | 路径 | 关键章节 |
|---|---|---|
| HLD | docs/design/HLD-PTM-TE-EXEC.md (v1.6) | §12.1 case_runner 执行流程；§12.2 _build_exec_env + execute_op 流程；§12.3 环境解析层设计；§13 NFR；§14 风险矩阵 RA-013/RA-014；Gotchas #1/#2/#3/#7/#8/#10/#11/#12/#13 |
| ADR | docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md | ADR-01（import 集成）；ADR-02（dry-run 默认门）；ADR-05（TREX_API_URL 注入，方案 a 扩展签名，环境文件优先）；ADR-08（TG 数据归属，topology yaml 用途扩展）；ADR-09（环境文件驱动） |
| Feature 矩阵 | docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md | FE-EX-02 case-execution 引擎；ST-EX-04 trigger_reasons（新建 case_runner.py/跨模块 import/三入口+取址+解析/case-execution 安装验证/load_env_file） |
| 开发计划 | process/DEVELOPMENT-PLAN.yaml | ST-EX-04 字段：depends_on=ST-EX-03(hard)/lld_gate=full-lld/dev_gate=cp5-approved/file_ownership 6 锚点 |
| 现有代码 | skills/policy-route-execution/scripts/op_mapper.py | execute_op(op_id,args,base_url,session_file,*,step_name,dry_run,authorized,timeout,username,password_env,exec_log_path,diag_snapshot_ref,step_id,step_refs_dir) 既有签名；_build_exec_env(base_url) 既有签名（ST-EX-03 扩展）；build_command/resolve_step_refs/handle_rollback 既有 |
| 现有代码 | script/ptm_team/install.py | PTM_TE_SKILLS 列表（第 101-106 行，当前 4 skill）；get_agent_skills 第 429-437 行；render_ptm_te_rule_body 第 460-478 行；check_installed_drift 第 1439 行 |

### 1.2 Story 范围与边界

**ST-EX-04 承载**（本 LLD 设计范围）：
- 新建 `skills/case-execution/SKILL.md`（skill 描述 + 使用说明 + 安全约束）
- 新建 `skills/case-execution/scripts/case_runner.py` 核心编排骨架：`main`/`discover_cases`/`parse_case_file`/`resolve_addresses`/`load_env_file`
- 修改 `script/ptm_team/install.py#PTM_TE_SKILLS`：新增 `"case-execution"` 条目
- case-execution 安装验证：install.py 安装后验证 skill 存在（Gotcha #10）
- TREX_API_URL 消费契约：case_runner 从 devices.yaml 读 tg.api_server，调 execute_op 时传 tg_api_server（env_topology ${ENV.tg.url} 优先，devices.yaml fallback）
- preconfigure_dut_interfaces 编排调用：main 流程在 login 后、用例主体前调用（R-F-028，--execute 模式），清理顺序编排

**ST-EX-04 不承载**（显式排除，由对应 Story LLD 承载）：
- `_build_exec_env` 签名扩展与 TREX_API_URL 注入实现 → ST-EX-03（op_mapper.py#_build_exec_env / #resolve_env_refs）
- `resolve_env_refs` 实现 → ST-EX-03（op_mapper.py#resolve_env_refs）
- `build_env_topology` 完整性校验 → ST-EX-17（case_runner.py#build_env_topology）
- `preconfigure_dut_interfaces` 实现细节 → ST-EX-17（case_runner.py#preconfigure_dut_interfaces）
- dry-run/授权门 `execute_steps`/`authorize` → ST-EX-05
- 逆序清理 `run_cleanup` → ST-EX-06
- fw_logout `cleanup_session` → ST-EX-07
- warming_up `apply_warming_up` → ST-EX-08
- `generate_report`/`diagnose_failure` → ST-EX-11
- install.py 规则块文本（render_ptm_te_rule_body ≥4 条新规则）→ ST-EX-02

### 1.3 关键约束（来自 HLD §1 约束 + 任务约束）

| 约束 ID | 内容 | 来源 |
|---|---|---|
| R-C-001 | 不改 ptm-atomic CLI 本体 | HLD §1 |
| R-C-002 | 不引入 pytest/robot/外部 eval 框架 | HLD §1 |
| R-C-003 | 不为 devices.yaml 引入 pydantic | HLD §1 |
| R-C-007 | skill 源在 ptm-team canonical，install.py 安装回 ptm-te workspace | HLD §1 |
| ptm-atomic 约束 | TG 操作经 `ptm-atomic run tg trex <action>`，框架禁止直接调 TG REST API；_build_exec_env 只注入 TREX_API_URL 环境变量供 ptm-atomic 子进程消费 | ADR-05/09，Gotcha #13，任务约束 |
| NO_CREDENTIAL_READ | 不读取凭据/secret/账户 | HLD §13.4，任务安全约束 |
| NO_PRODUCTION_WRITE | dry-run 默认门，--execute 需授权 | HLD §13.4，ADR-02 |
| NO_EXTERNAL_PUBLISH | 不发布到外部 | 任务安全约束 |
| 设计通过不等于运行授权 | CP5 设计确认不授权 --execute 运行 | 任务安全约束 |

## 2. 文件影响范围

### 2.1 新建文件

| 文件路径 | 类型 | 锚点归属 | 说明 |
|---|---|---|---|
| skills/case-execution/SKILL.md | 新建 | ST-EX-04#SKILL.md | skill 描述 + 使用说明 + 安全约束；frontmatter（name/description）；dry-run 默认门说明；三入口使用示例；安全约束段（NO_CREDENTIAL_READ/NO_PRODUCTION_WRITE/NO_EXTERNAL_PUBLISH） |
| skills/case-execution/scripts/case_runner.py | 新建 | ST-EX-04#main/#discover_cases/#parse_case_file/#resolve_addresses | 编排层主模块；ST-EX-04 实现 main/discover_cases/parse_case_file/resolve_addresses/load_env_file 函数；ST-EX-05/06/07/08/11/17 后续填充 execute_steps/run_cleanup/cleanup_session/apply_warming_up/generate_report/build_env_topology/preconfigure_dut_interfaces 等函数 |

### 2.2 修改文件

| 文件路径 | 修改锚点 | 修改内容 | 归属 |
|---|---|---|---|
| script/ptm_team/install.py | PTM_TE_SKILLS（第 101-106 行） | 列表新增 `"case-execution"` 条目 | ST-EX-04#PTM_TE_SKILLS |

### 2.3 不修改文件（显式排除）

| 文件路径 | 原因 | 归属 Story |
|---|---|---|
| skills/policy-route-execution/scripts/op_mapper.py | _build_exec_env 签名扩展 + resolve_env_refs 实现 | ST-EX-03 |
| script/ptm_team/install.py#render_ptm_te_rule_body | 规则块文本 ≥4 条新规则 | ST-EX-02 |
| skills/case-execution/scripts/case_runner.py#build_env_topology | env_topology 完整性校验 | ST-EX-17 |
| skills/case-execution/scripts/case_runner.py#preconfigure_dut_interfaces | DUT 接口预配置实现 | ST-EX-17 |
| skills/case-execution/scripts/case_runner.py#execute_steps | dry-run/授权门 | ST-EX-05 |
| skills/case-execution/scripts/case_runner.py#run_cleanup | 逆序清理 | ST-EX-06 |
| skills/case-execution/scripts/case_runner.py#cleanup_session | fw_logout 登出 | ST-EX-07 |

### 2.4 file_ownership 锚点声明

本 Story 独占写入以下锚点（DEVELOPMENT-PLAN.yaml 声明，CP4 PASS 已校验文件无冲突）：

| 锚点 | 写入内容 |
|---|---|
| skills/case-execution/SKILL.md | 全文新建 |
| skills/case-execution/scripts/case_runner.py#main | main(argv) 函数实现 |
| skills/case-execution/scripts/case_runner.py#discover_cases | discover_cases 函数实现 |
| skills/case-execution/scripts/case_runner.py#parse_case_file | parse_case_file 函数实现 |
| skills/case-execution/scripts/case_runner.py#resolve_addresses | resolve_addresses 函数实现 |
| script/ptm_team/install.py#PTM_TE_SKILLS | 列表新增 case-execution |

**共享文件说明**：`skills/case-execution/scripts/case_runner.py` 是多 Story 共享文件（FE-EX-02 12 Story 共享）。ST-EX-04 只写上述 4 个函数锚点 + 模块级常量/imports/sys.path 注入；其余函数锚点由对应 Story 写入。CP4 已校验各 Story file_ownership 无冲突（不同函数锚点）。

## 3. 接口设计

### 3.1 CLI 入口（main）

```python
def main(argv: Optional[List[str]] = None) -> int:
    """case_runner CLI 入口。

    命令格式（HLD §12.1）：
        case_runner.py run --cases-dir <dir> --devices-yaml <path>
            [--topology-yaml <path>] [--env-file <path>]
            [--execute --authorized] [--tag <tags>] [--keyword <kw>]
            [--case-file <path>] [--password-env <ENV_VAR>]

    三入口（SM-EX-02）：
      --case-file <path>    单用例执行调试（UC-EX-03）
      --cases-dir <dir>     目录 glob 批量执行（UC-EX-02）
      （未指定 case-file/cases-dir 时报错）

    可选参数：
      --topology-yaml <path>  topology yaml 路径（ADR-08，用例含 TG step 时必需）
      --env-file <path>       环境文件路径（ADR-09，用例含 ${ENV.*} 时必需；
                              缺省时复用 --topology-yaml 作为 env_topology 源，见 LCQ-ST-EX-04-01）
      --execute --authorized  真实执行模式（默认 dry-run，ADR-02）
      --tag <tags>            按 frontmatter tags 列精确过滤（ST-EX-16 复用）
      --keyword <kw>          按 frontmatter 关键词列模糊匹配（ST-EX-16 复用）
      --password-env <ENV>    Web 密码环境变量名（默认 FW_WEB_PASSWORD）

    Returns:
      0 成功；非 0 失败（启动校验失败/解析失败/执行错误）
    """
```

**argparse 子命令设计**：采用 `run` 子命令（`case_runner.py run ...`），为后续扩展（如 `validate`/`list`）预留空间。`run` 子命令参数如上。

**入口选择优先级**：`--case-file` 优先于 `--cases-dir`；同时指定时 `--case-file` 生效（单用例调试场景）。

### 3.2 用例发现（discover_cases）

```python
def discover_cases(
    cases_dir: str,
    tag: Optional[List[str]] = None,
    keyword: Optional[str] = None,
) -> List[Path]:
    """扫描目录下全部 .md 用例文件，可选按 tag/keyword 过滤。

    Args:
        cases_dir: 用例目录（如 cases/IPv4策略路由/配置管理/策略配置/）
        tag: frontmatter tags 列精确过滤（ST-EX-16）；None 不过滤
        keyword: frontmatter 关键词列模糊匹配（ST-EX-16）；None 不过滤

    Returns:
        排序后的 .md 文件 Path 列表（按文件名排序，保证执行顺序确定性）

    行为：
      - glob 递归扫描 **/*.md
      - 排除 README.md（目录说明文件，非用例）
      - tag/keyword 过滤需读取 frontmatter（parse_case_file 解析后判断）；
        ST-EX-04 实现基本 glob + README 排除，tag/keyword 过滤逻辑由 ST-EX-16 填充
        （ST-EX-04 预留 filter_by_tag/filter_by_keyword 调用点，ST-EX-16 实现）
    """
```

**编号正则匹配**（ADR-07，Gotcha #6）：文件名 `<编号>-<名称>.md`，编号格式 `PC-[A-Z0-9]+-\d+-\d+-\d+`。discover_cases 按 `^(PC-[A-Z0-9]+-\d+-\d+-\d+)` 正则提取编号前缀排序，避免连字符冲突。

### 3.3 用例解析（parse_case_file）

```python
# frontmatter 编号正则（ADR-07）
CASE_ID_RE = re.compile(r"^(PC-[A-Z0-9]+-\d+-\d+-\d+)")

def parse_case_file(path: Path) -> dict:
    """解析单个用例 .md 文件，返回结构化用例对象。

    用例文件结构：
      --- frontmatter (YAML) ---
      用例编号: PC-M4-01-01
      用例名称: 创建策略路由-有效参数
      标签: [IPv4, 策略路由, P0]
      关键词: 策略路由/有效参数
      ... (frontmatter 16 列，AGA-03=C: case_runner 忽略测试步骤/预期结果冗余列)
      --- end frontmatter ---

      ## case_steps
      ```yaml
      - step_id: STEP-001
        step_name: 登录Web管理
        op_id: fw_login_web_management
        args: {username: "admin"}
        target: dut
      - step_id: STEP-002
        step_name: 配置策略路由
        op_id: fw_config_policy_route
        args: {source_network: "OBJ-SRC-192", ...}
        target: dut
      ```

    Args:
        path: 用例文件路径

    Returns:
        dict: {
            "case_id": "PC-M4-01-01",        # frontmatter 用例编号（唯一标识，ADR-07）
            "case_name": "创建策略路由-...",   # frontmatter 用例名称
            "tags": ["IPv4", ...],            # frontmatter 标签列
            "keyword": "策略路由/...",         # frontmatter 关键词列
            "frontmatter": {...},             # 完整 frontmatter dict（16 列，含冗余列保留但不消费）
            "case_steps": [...],              # case_steps YAML body 解析为 list[dict]
            "source_path": str(path),
        }

    Raises:
        CaseParseError: frontmatter 缺失/编号缺失/case_steps YAML 解析失败
    """
```

**frontmatter 16 列处理**（AGA-03=C，Gotcha #7）：case_runner 只消费编号/名称/tags/关键词 4 列；测试步骤/预期结果列与 case_steps YAML 冗余，解析时忽略（不校验内容一致性）。完整 frontmatter 保留在返回 dict 的 `frontmatter` 字段供审计，但执行只读 `case_steps`。16 列的完整列名约定由 ST-EX-15（用例结构化约定）承载。

**case_steps YAML 字段**（ST-EX-04 解析，字段语义由 ST-EX-05/06/08/10/11 消费）：

| 字段 | 类型 | 消费 Story | 说明 |
|---|---|---|---|
| step_id | str | ST-EX-04/06 | STEP-001，step-refs 落盘键 |
| step_name | str | ST-EX-04 | 步骤名，写入 envelope |
| op_id | str | ST-EX-04 | 原子操作 ID，映射 op_mapper |
| args | dict | ST-EX-04 | 参数 dict，可含 ${ENV.*}/${STEP-N.id} |
| target | str | ST-EX-04 | "dut"/"tg"，决定 timeout（Gotcha #9） |
| known_issue | bool | ST-EX-10 | KNOWN_FAIL 标记 |
| warming_up | bool | ST-EX-08 | ARP 预热标记，强制 post_op |
| post_op | dict | ST-EX-08 | warming_up 配套清理 op |
| retry | dict | ST-EX-09 | 轮询配置 |

ST-EX-04 只解析 YAML 为 list[dict]，字段语义消费由各 Story 承载。

### 3.4 设备取址（resolve_addresses）

```python
def resolve_addresses(devices_yaml_path: str) -> dict:
    """从 devices.yaml 读取 DUT 和 TG 设备地址。

    Args:
        devices_yaml_path: devices.yaml 路径

    Returns:
        dict: {
            "dut_url": "https://10.113.55.51",      # firewall.host（DUT Web 管理地址）
            "tg_api_server": "10.113.52.253:8450",   # tg.api_server（TREX_API_URL fallback，ADR-05）
            "dut_username": "admin",                 # firewall.username（默认 admin）
            "dut_password_env": "FW_WEB_PASSWORD",   # firewall.password_env（默认 FW_WEB_PASSWORD）
        }

    Raises:
        DeviceConfigError: devices.yaml 缺失/firewall.host 缺失/tg.api_server 缺失（DA-003）
    """
```

**取址逻辑**（DA-003，ADR-08）：
- DUT_URL = `devices.yaml.firewall.host`（必需，缺失报 DeviceConfigError）
- TG_API_SERVER = `devices.yaml.tg.api_server`（TREX_API_URL fallback 源；缺失时若用例不含 TG step 则不报错，含 TG step 则报错）
- 凭据用 `${ENV_VAR}` 占位或 `--password-env` 传环境变量名（ADR-02，R-C-007，NO_CREDENTIAL_READ）；resolve_addresses 不读取密码值，只返回环境变量名

**TREX_API_URL 来源优先级**（ADR-05，HLD §12.2/§12.3）：
1. 环境文件 `${ENV.tg.url}`（= env_topology.nodes.tg1.trex_api_url）优先
2. devices.yaml tg.api_server fallback

resolve_addresses 只负责读取 devices.yaml 的 tg.api_server（fallback 源）；${ENV.tg.url} 优先级由 case_runner main 在调用 execute_op 时决定（若 env_topology 含 nodes.tg1.trex_api_url 则优先用之，否则用 resolve_addresses 返回的 tg_api_server）。

### 3.5 环境文件加载（load_env_file）

```python
def load_env_file(env_file_path: Optional[str]) -> Optional[dict]:
    """加载 --env-file 为 env_topology dict。

    env_topology 契约（HLD §12.3，ADR-09）：
      {
        "port_mapping": {                    # 逻辑端口 -> 物理映射
          "port1": {"tg": "1/1/1", "dut": "eth0"},
          "port2": {"tg": "1/1/2", "dut": "eth1"}
        },
        "nodes": {                           # 设备节点元数据 + 接口 IP/网关/下一跳
          "tg1": {"trex_api_url": "...", "interfaces": {"port1": {"ip": "...", "gw": "..."}}},
          "dut1": {"host": "...", "next_hop": "...", "interfaces": {"port1": {"ip": "..."}}}
        },
        "links": [                           # 组网连接拓扑
          {"from": "tg1.port1", "to": "dut1.port1", "name": "link1"}
        ]
      }

    Args:
        env_file_path: --env-file 路径；None 时返回 None（向后兼容，无 ${ENV.*} 用例）

    Returns:
        env_topology dict；env_file_path 为 None 时返回 None

    Raises:
        EnvFileError: 文件不存在/YAML 解析失败/port_mapping/nodes/links 字段缺失（DA-006）

    边界：
      - load_env_file 只做 YAML 加载 + 顶层三字段（port_mapping/nodes/links）存在性校验
      - port_mapping 完整性校验（覆盖全部逻辑端口，DA-007）由 ST-EX-17 build_env_topology 承载
      - ${ENV.*} 占位符解析由 ST-EX-03 resolve_env_refs 承载（op_mapper.execute_op 内部）
    """
```

**--env-file 与 --topology-yaml 关系**（LCQ-ST-EX-04-01，非阻塞）：
- `--env-file` 缺省时，若 `--topology-yaml` 指定的 topology yaml 含 port_mapping/nodes/links 字段，则复用 topology yaml 作为 env_topology 源（HLD §12.3"暂用 topology yaml 作环境文件载体"）
- `--env-file` 显式指定时，优先用 `--env-file`（支持环境文件与组网拓扑分离的场景）
- load_env_file 接收的 env_file_path 已由 main 决定（main 逻辑：env_file = args.env_file or args.topology_yaml）

**YAML 1.1 整数陷阱**（Gotcha #11）：load_env_file 加载后，port_mapping 中物理端口名必须为字符串。ST-EX-04 在 load_env_file 中做基本类型校验（端口名非字符串时报 EnvFileError）；完整校验由 ST-EX-17 build_env_topology 承载。

### 3.6 与 op_mapper.execute_op 的消费契约

ST-EX-04 是 op_mapper.execute_op 的消费方。ST-EX-03 扩展 execute_op 签名后，ST-EX-04 调用契约如下：

```python
# ST-EX-03 扩展后的 execute_op 签名（ST-EX-04 消费，不实现）
envelope = op_mapper.execute_op(
    op_id=step["op_id"],
    args=step["args"],
    base_url=addresses["dut_url"],
    session_file=session_file,
    step_name=step["step_name"],
    dry_run=dry_run,              # ST-EX-05 授权门决定
    authorized=authorized,        # ST-EX-05 授权门决定
    timeout=timeout,              # ST-EX-04 按 target 设置（Gotcha #9: TG 90s / DUT 45s）
    username=addresses["dut_username"],
    password_env=addresses["dut_password_env"],
    exec_log_path=exec_log_path,
    step_id=step["step_id"],
    step_refs_dir=step_refs_dir,
    env_topology=env_topology,    # ST-EX-03 新增参数，ST-EX-04 传入 load_env_file 结果
    tg_api_server=tg_api_server,  # ST-EX-03 新增参数，ST-EX-04 传入（env_topology ${ENV.tg.url} 优先，devices.yaml fallback）
)
```

**tg_api_server 解析逻辑**（ST-EX-04 在 main 中实现，传给 execute_op）：

```python
def _resolve_tg_api_server(env_topology: Optional[dict], devices_tg_api_server: str) -> str:
    """TREX_API_URL 来源解析（ADR-05 环境文件优先）。

    优先级：
      1. env_topology.nodes.tg1.trex_api_url（${ENV.tg.url}，环境文件优先）
      2. devices_tg_api_server（devices.yaml tg.api_server fallback）
    """
    if env_topology and env_topology.get("nodes", {}).get("tg1", {}).get("trex_api_url"):
        return env_topology["nodes"]["tg1"]["trex_api_url"]
    return devices_tg_api_server
```

**ptm-atomic 约束**（Gotcha #13，ADR-05/09）：ST-EX-04 只做参数解析与 tg_api_server 传入；TREX_API_URL 环境变量注入由 op_mapper._build_exec_env 完成（ST-EX-03 实现）；TG 操作经 ptm-atomic run tg trex <action> 原子操作下发，框架禁止直接调 TG REST API。code review 检查 case_runner.py 无 requests/urllib 直接调 TG 的代码。

### 3.7 与下游 Story 的接口边界

| 下游 Story | 接口函数 | ST-EX-04 提供什么 | 下游 Story 实现什么 |
|---|---|---|---|
| ST-EX-05 | execute_steps/authorize | main 调用点 + dry_run/authorized 传参骨架 | dry-run 默认门 + --execute 授权门 + runtime_authorization 审计 |
| ST-EX-06 | run_cleanup | main 调用点（用例后逆序清理） | 逆序清理 mutation ops + step-refs 读取 + handle_rollback 调用 |
| ST-EX-07 | cleanup_session | main 调用点（finally 块 fw_logout） | fw_logout op 映射 + 降级 session 文件清理 |
| ST-EX-08 | apply_warming_up | main 在 step 执行循环中调用点 | warming_up:true 强制 post_op 引擎逻辑 |
| ST-EX-11 | generate_report/diagnose_failure | main 调用点（全部用例后） | 失败诊断字段提取 + report.md 模板 + 幂等容错记录 |
| ST-EX-17 | build_env_topology/preconfigure_dut_interfaces | main 调用点 + env_topology 传入 | env_topology 完整性校验 + DUT 接口预配置 + 用例后清理 |

**编排骨架原则**：ST-EX-04 的 main 实现"调用什么、何时调用、传什么参数"的编排骨架；下游 Story 填充被调函数的实现。ST-EX-04 main 中的下游调用点以函数调用形式预留（如 `run_cleanup(...)`），下游 Story 实现函数体。CP5 前各 Story LLD 独立确认，CP6 实现时按 Wave 顺序填充。

## 4. 核心流程

### 4.1 case_runner 执行流程（对应 HLD §12.1 七步）

```
case_runner.py run --cases-dir <dir> --devices-yaml <path> [--env-file <path>] [--topology-yaml <path>] [--execute --authorized] [--tag] [--keyword]
  │
  ├─ [1] 启动校验（_validate_startup）
  │   ├─ which ptm-atomic（DA-001）；不在 PATH 报 StartupError
  │   ├─ devices.yaml 含 firewall.host（DA-003）；缺失报 DeviceConfigError
  │   ├─ devices.yaml 含 tg.api_server（DA-003）；用例含 TG step 时必需，缺失报错
  │   ├─ --topology-yaml 存在（若用例含 TG step，ADR-08）；缺失报错
  │   └─ --env-file 存在且含 port_mapping/nodes/links（DA-006/007，若用例含 ${ENV.*}，ADR-09）
  │      └─ --env-file 缺省时复用 --topology-yaml（LCQ-ST-EX-04-01 方案 A）
  │
  ├─ [2] 设备准备 + 环境加载
  │   ├─ addresses = resolve_addresses(devices_yaml)  # DUT_URL + TG_API_SERVER fallback
  │   ├─ env_topology = load_env_file(env_file)       # --env-file 加载（ST-EX-04）
  │   ├─ build_env_topology(env_topology)             # 完整性校验（ST-EX-17 实现，ST-EX-04 调用点）
  │   ├─ tg_api_server = _resolve_tg_api_server(env_topology, addresses["tg_api_server"])  # ADR-05 优先级
  │   ├─ preconfigure_dut_interfaces(addresses, env_topology, authorized)  # R-F-028（ST-EX-17 实现，ST-EX-04 调用点）
  │   │   └─ --execute 模式按 nodes.dut1.interfaces 调 fw_update_interface；dry-run 跳过
  │   └─ 用例后清理顺序编排（HLD §12.3）：
  │       先 ST-EX-06 run_cleanup（mutation ops 逆序清理）
  │       再 ST-EX-17 preconfigure_dut_interfaces 逆序还原（框架预配置接口后还原）
  │
  ├─ [3] 用例发现
  │   ├─ --case-file: parse_case_file(path) 单用例（UC-EX-03）
  │   ├─ --cases-dir: discover_cases(dir, tag, keyword) glob 批量（UC-EX-02）
  │   ├─ --tag: 按 frontmatter tags 列精确过滤（ST-EX-16 实现，ST-EX-04 预留调用点）
  │   └─ --keyword: 按 frontmatter 关键词列模糊匹配（ST-EX-16 实现，ST-EX-04 预留调用点）
  │
  ├─ [4] 预登录（--execute 模式，dry-run 跳过）
  │   ├─ op_mapper.execute_op(fw_login_web_management, env_topology=env_topology, tg_api_server=tg_api_server)
  │   ├─ 建立 SHARED_SESSION（session_file 写入 ~/.local/state/ptm-atomic/）
  │   └─ ConnectTimeout 重试 3 次（15/20/25s 递增，HLD §13.3）
  │   └─ dry-run 模式：fw_login 等 mutation op 只构建命令打印，不实际登录（Gotcha #8）
  │
  ├─ [5] 逐用例执行
  │   ├─ for case_file in discovered_cases:
  │   │   ├─ case = parse_case_file(case_file)  # frontmatter 16 列 + case_steps YAML
  │   │   ├─ for step in case["case_steps"]:
  │   │   │   ├─ fw_login_web_management step 标记 SKIP（复用 SHARED_SESSION，Gotcha #8）
  │   │   │   ├─ timeout = 90 if step.target == "tg" else 45  # Gotcha #9
  │   │   │   ├─ envelope = op_mapper.execute_op(
  │   │   │   │       step.op_id, step.args, addresses["dut_url"], session_file,
  │   │   │   │       env_topology=env_topology, tg_api_server=tg_api_server, ...)
  │   │   │   │   └─ op_mapper 内部：resolve_env_refs -> resolve_step_refs -> validate_args -> build_command
  │   │   │   │   └─ tg_* op: _build_exec_env 注入 TREX_API_URL（ST-EX-03 实现）
  │   │   │   ├─ apply_warming_up(step, envelope)  # ST-EX-08 实现，ST-EX-04 调用点
  │   │   │   ├─ classify_result(step, envelope)   # ST-EX-10 实现，ST-EX-04 调用点
  │   │   │   └─ apply_retry(step, envelope)       # ST-EX-09 实现，ST-EX-04 调用点
  │   │   ├─ run_cleanup(case, step_refs_dir)      # ST-EX-06 实现，ST-EX-04 调用点（逆序清理 mutation ops）
  │   │   └─ 写 result.json（runs/<run-id>/result.json）
  │   ├─ 用例间 8s 间隔（HLD §13.1）
  │   └─ op 间 2s 间隔（HLD §13.1）
  │
  ├─ [6] fw_logout 会话清理（finally 块，--execute 模式）
  │   ├─ cleanup_session(session_file, addresses, env_topology, tg_api_server)  # ST-EX-07 实现
  │   ├─ op_mapper.execute_op(fw_logout, ...) 或降级 os.remove(session_file)（ADR-04，DQ-02）
  │   └─ result.json 含 logout 状态
  │
  ├─ [7] preconfigure_dut_interfaces 逆序还原（--execute 模式，finally 块）
  │   └─ ST-EX-17 实现，ST-EX-04 编排调用点（在 run_cleanup 之后，确保用例写的 op 先回滚）
  │
  └─ [8] 生成 report.md
      ├─ generate_report(results)  # ST-EX-11 实现，ST-EX-04 调用点
      ├─ 四态统计表（PASS/FAIL/KNOWN_FAIL/ERROR）
      └─ 失败 step 诊断（error_type/error_code/reason/details/command）
```

### 4.2 清理顺序编排（HLD §12.3，R-F-028）

**用例后清理顺序**（ST-EX-04 在 main 中编排，确保清理顺序正确）：

```
每用例结束:
  1. ST-EX-06 run_cleanup(case, step_refs_dir)
     └─ 逆序清理 case_steps 中的 mutation ops（用例写的 op 先回滚）
  2. （无框架预配置接口清理，框架预配置在全部用例后统一还原）

全部用例结束 (finally):
  1. ST-EX-07 cleanup_session(session_file)
     └─ fw_logout 登出
  2. ST-EX-17 preconfigure_dut_interfaces 逆序还原
     └─ 框架预配置的 DUT 接口后还原（确保用例写的 op 先回滚，框架预配置的接口后还原）
```

**顺序保证**：先执行用例级 mutation ops 逆序清理（ST-EX-06），再执行框架级预配置接口逆序还原（ST-EX-17）。这确保用例创建的临时对象（如策略路由规则）先被删除，框架预配置的接口 IP 后被还原，避免接口还原时仍有依赖该接口的残留对象。

### 4.3 dry-run 模式行为（ADR-02，Gotcha #8）

| 步骤 | dry-run 模式 | --execute 模式 |
|---|---|---|
| [1] 启动校验 | 执行（含 ptm-atomic/devices.yaml 校验） | 执行 |
| [2] 设备准备 | load_env_file 执行；preconfigure_dut_interfaces 跳过 | 全执行 |
| [3] 用例发现 | 执行 | 执行 |
| [4] 预登录 | 跳过（不建立 SHARED_SESSION）；fw_login step 只构建命令打印 | 执行（建立 SHARED_SESSION） |
| [5] 逐用例执行 | execute_op(dry_run=True)；mutation op 只构建命令打印 | execute_op(dry_run=False, authorized=True) |
| [6] fw_logout | 跳过 | 执行 |
| [7] 接口还原 | 跳过 | 执行 |
| [8] report | 执行（dry-run 结果） | 执行 |

**dry-run 下 resolve_env_refs 仍执行**（Gotcha #12）：验证 ${ENV.*} 占位符解析正确性，仅跳过 mutation op 实际执行。ST-EX-04 在 dry-run 模式仍传 env_topology 给 execute_op，op_mapper 内部 resolve_env_refs 照常执行（ST-EX-03 实现）。

## 5. 异常处理

### 5.1 异常分类与错误码

| 异常类型 | 触发条件 | 错误码 | 处理行为 | 归属 |
|---|---|---|---|---|
| StartupError | which ptm-atomic 失败（DA-001） | STARTUP_PTM_ATOMIC_NOT_FOUND | main 返回 2，打印提示安装 ptm-atomic | ST-EX-04 |
| DeviceConfigError | devices.yaml 缺失/firewall.host 缺失/tg.api_server 缺失（DA-003） | DEVICE_CONFIG_MISSING | main 返回 3，打印缺失字段 | ST-EX-04 |
| EnvFileError | --env-file 不存在/YAML 解析失败/port_mapping/nodes/links 缺失（DA-006） | ENV_FILE_INVALID | main 返回 4，打印缺失字段 | ST-EX-04 |
| CaseParseError | frontmatter 缺失/编号缺失（ADR-07）/case_steps YAML 解析失败 | CASE_PARSE_FAILED | 该用例标记 ERROR，继续执行下一用例 | ST-EX-04 |
| ImportError | import op_mapper 失败（Gotcha #2，sys.path 问题） | IMPORT_OP_MAPPER_FAILED | main 返回 5，打印 sys.path 调试信息 | ST-EX-04 |
| TimeoutError | TG op 90s / DUT op 45s 超时（Gotcha #9） | STEP_TIMEOUT | envelope error_type=EXEC_FAILED；TG 重试 3 次 / DUT 重试 1 次 | ST-EX-04（timeout 设置）/ ST-EX-05（重试编排） |
| StateInvalidError | STATE_INVALID（session 失效） | STATE_INVALID | op_mapper _reconnect_and_retry 1 次（既有逻辑） | op_mapper |
| OpNotFoundError | op_id 未在映射表 | OP_NOT_FOUND | envelope error_type=OP_NOT_FOUND；step 标记 ERROR | op_mapper（既有） |

### 5.2 用例级容错（不中断批量执行）

**批量执行容错**：单用例 parse_case_file 失败（CaseParseError）或单 step 执行失败（envelope status=error），不中断批量执行。失败用例/step 记入 result.json，继续执行下一用例。仅以下情况中断批量执行：
- 启动校验失败（StartupError/DeviceConfigError/EnvFileError/ImportError）
- 预登录失败（--execute 模式，SHARED_SESSION 无法建立）

### 5.3 import op_mapper 路径处理（Gotcha #2）

```python
# case_runner.py 顶部（Gotcha #2 规避）
import sys
import os
_SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
_OP_MAPPER_DIR = os.path.normpath(os.path.join(_SCRIPT_DIR, "../../policy-route-execution/scripts"))
if _OP_MAPPER_DIR not in sys.path:
    sys.path.insert(0, _OP_MAPPER_DIR)
import op_mapper  # noqa: E402
```

**路径解析**：case_runner.py 在 `skills/case-execution/scripts/`，op_mapper.py 在 `skills/policy-route-execution/scripts/`。相对路径 `../../policy-route-execution/scripts` 跨 skill 目录。install.py 安装到 workspace 后两个 skill 的 scripts 目录在同一 `.claude/skills/` 下，相对路径保持有效。

**失败行为**：import op_mapper 失败时 main 捕获 ImportError，返回错误码 5，打印 sys.path 和 _OP_MAPPER_DIR 调试信息，提示用户检查 skill 安装完整性。

## 6. 测试设计

### 6.1 验证方式声明

| 验证层 | 方式 | 说明 |
|---|---|---|
| 单元测试 | Python unittest（不引入 pytest，R-C-002） | mock op_mapper.execute_op，测试 discover_cases/parse_case_file/resolve_addresses/load_env_file |
| dry-run 校验 | case_runner --dry-run 批量校验 | 24 用例 dry-run 全部解析成功（DA-002） |
| 静态审查 | code review + grep | 无硬编码 IP（SM-EX-03）；无 requests/urllib 直接调 TG（Gotcha #13） |
| 集成测试 | --execute 模式（需 runtime_authorization） | CP7 采用 static-only + dry-run（DQ-01 推荐）；--execute 集成测试需独立运行授权 |

**validation_mode 声明**：ST-EX-04 验证模式为 `static-only` + `dry-run-only`（CP7 不执行真实设备写操作，需 runtime_authorization 独立授权）。性能指标（24 用例 < 5 分钟，HLD §13.1）在 static-only 下不可直接验证，需 --execute 集成测试。

### 6.2 单元测试用例

| 测试 ID | 测试对象 | 测试场景 | 预期结果 |
|---|---|---|---|
| UT-EX-04-01 | discover_cases | 目录含 3 个 .md + 1 个 README.md | 返回 3 个 Path，排除 README.md |
| UT-EX-04-02 | discover_cases | 空目录 | 返回空列表 |
| UT-EX-04-03 | discover_cases | 按 tag 过滤 | 只返回含指定 tag 的用例（ST-EX-16 协作） |
| UT-EX-04-04 | parse_case_file | 合法 frontmatter + case_steps | 返回 case_id/case_name/tags/case_steps |
| UT-EX-04-05 | parse_case_file | frontmatter 缺失 | 抛 CaseParseError |
| UT-EX-04-06 | parse_case_file | 编号缺失（ADR-07） | 抛 CaseParseError |
| UT-EX-04-07 | parse_case_file | case_steps YAML 语法错误 | 抛 CaseParseError |
| UT-EX-04-08 | parse_case_file | frontmatter 含 16 列（含冗余测试步骤/预期结果） | 忽略冗余列，只消费 4 列（AGA-03=C） |
| UT-EX-04-09 | resolve_addresses | devices.yaml 含 firewall.host + tg.api_server | 返回 dut_url + tg_api_server |
| UT-EX-04-10 | resolve_addresses | devices.yaml 缺 firewall.host | 抛 DeviceConfigError（DA-003） |
| UT-EX-04-11 | resolve_addresses | devices.yaml 缺 tg.api_server（用例含 TG step） | 抛 DeviceConfigError |
| UT-EX-04-12 | load_env_file | 合法 env-file 含 port_mapping/nodes/links | 返回 env_topology dict |
| UT-EX-04-13 | load_env_file | env_file_path=None | 返回 None（向后兼容） |
| UT-EX-04-14 | load_env_file | port_mapping 缺失 | 抛 EnvFileError（DA-006） |
| UT-EX-04-15 | load_env_file | 端口名为整数（YAML 1.1 陷阱，Gotcha #11） | 抛 EnvFileError |
| UT-EX-04-16 | _resolve_tg_api_server | env_topology 含 nodes.tg1.trex_api_url | 返回 env_topology 值（环境文件优先，ADR-05） |
| UT-EX-04-17 | _resolve_tg_api_server | env_topology 无 trex_api_url | 返回 devices fallback（ADR-05） |
| UT-EX-04-18 | _resolve_tg_api_server | env_topology=None | 返回 devices fallback |
| UT-EX-04-19 | main | which ptm-atomic 失败 | 返回 2（StartupError） |
| UT-EX-04-20 | main | import op_mapper 失败（Gotcha #2） | 返回 5（ImportError） |
| UT-EX-04-21 | main | dry-run 模式不预登录（Gotcha #8） | 无 SHARED_SESSION，fw_login step 只构建命令 |
| UT-EX-04-22 | timeout | step.target="tg" | timeout=90（Gotcha #9） |
| UT-EX-04-23 | timeout | step.target="dut" | timeout=45（Gotcha #9） |

### 6.3 dry-run 校验用例

| 测试 ID | 场景 | 预期 |
|---|---|---|
| DR-EX-04-01 | 24 用例 dry-run 批量执行 | 全部解析成功，0 硬编码 IP（SM-EX-03） |
| DR-EX-04-02 | dry-run 含 ${ENV.*} 用例 | resolve_env_refs 解析成功（Gotcha #12，向后兼容） |
| DR-EX-04-03 | dry-run 无 ${ENV.*} 用例（字面值） | 原样透传（Gotcha #12） |
| DR-EX-04-04 | dry-run fw_login step | 只构建命令打印，不实际登录（Gotcha #8） |

### 6.4 安装验证用例（Gotcha #10）

| 测试 ID | 场景 | 预期 |
|---|---|---|
| INST-EX-04-01 | install.py install ptm-te --component full | .claude/skills/case-execution/ 存在 |
| INST-EX-04-02 | install -> uninstall -> install 循环 | case-execution skill 不丢失（Gotcha #10） |
| INST-EX-04-03 | check_installed_drift（ptm-te） | case-execution skill 无漂移 |
| INST-EX-04-04 | PTM_TE_SKILLS 含 case-execution | get_agent_skills("ptm-te") 返回含 case-execution |

## 7. 实施步骤

### 7.1 实现顺序（ST-EX-04 内部）

| 步骤 | 内容 | 依赖 | 验证 |
|---|---|---|---|
| S1 | 创建 skills/case-execution/ 目录结构（SKILL.md + scripts/） | 无 | 目录存在 |
| S2 | 实现 case_runner.py 模块骨架：imports + sys.path 注入（Gotcha #2）+ 异常类定义 + 模块常量 | S1 | import op_mapper 成功 |
| S3 | 实现 resolve_addresses(devices_yaml_path) | S2 | UT-EX-04-09..11 |
| S4 | 实现 load_env_file(env_file_path) | S2 | UT-EX-04-12..15 |
| S5 | 实现 _resolve_tg_api_server(env_topology, devices_tg_api_server) | S4 | UT-EX-04-16..18 |
| S6 | 实现 parse_case_file(path)（frontmatter + case_steps YAML 解析，AGA-03=C） | S2 | UT-EX-04-04..08 |
| S7 | 实现 discover_cases(cases_dir, tag, keyword)（glob + 编号正则，ADR-07） | S6 | UT-EX-04-01..03 |
| S8 | 实现 main(argv) 编排骨架：argparse + 启动校验 + 调用点预留（下游 Story 函数调用） | S3,S4,S5,S6,S7 | UT-EX-04-19..23 |
| S9 | 修改 install.py PTM_TE_SKILLS 新增 "case-execution" | S1 | INST-EX-04-04 |
| S10 | 编写 SKILL.md（skill 描述 + 使用说明 + 安全约束） | S8 | SKILL.md frontmatter 合法 |
| S11 | 编写单元测试（unittest，不引入 pytest） | S3..S8 | UT-EX-04-01..23 全通过 |
| S12 | dry-run 校验（24 用例，需 ST-EX-13 用例就绪后） | S8, ST-EX-13 | DR-EX-04-01..04 |
| S13 | 安装验证循环（install/uninstall/install，Gotcha #10） | S9 | INST-EX-04-01..04 |

### 7.2 与下游 Story 的实现协作

ST-EX-04 实现编排骨架后，下游 Story 按 Wave 顺序填充被调函数：

| Wave | Story | 填充函数 | ST-EX-04 预留调用点 |
|---|---|---|---|
| 2 | ST-EX-05 | execute_steps/authorize | main [5] step 执行循环 |
| 2 | ST-EX-06 | run_cleanup | main [5] 用例后清理 |
| 2 | ST-EX-07 | cleanup_session | main [6] finally fw_logout |
| 3 | ST-EX-08 | apply_warming_up | main [5] step 执行循环 |
| 3 | ST-EX-10 | classify_result | main [5] step 执行循环 |
| 3 | ST-EX-11 | generate_report/diagnose_failure | main [8] report 生成 |
| 3 | ST-EX-17 | build_env_topology/preconfigure_dut_interfaces | main [2] 环境加载/预配置 + [7] 接口还原 |

**实现协作原则**：ST-EX-04 的 main 中下游调用点以函数调用形式预留（如 `run_cleanup(case, step_refs_dir)`）；下游 Story 实现函数体时不得修改 main 的调用签名和调用位置，只填充函数实现。若下游 Story 需调整调用契约，须通过 CR 修改 ST-EX-04 LLD。

## 8. 回滚策略

### 8.1 代码回滚

| 回滚场景 | 回滚动作 | 影响 |
|---|---|---|
| case_runner.py 实现缺陷 | git revert ST-EX-04 commit；case-execution skill 未安装前无影响 | 无（新建文件，回滚后等同于未实现） |
| install.py PTM_TE_SKILLS 修改缺陷 | git revert 该行修改；PTM_TE_SKILLS 恢复 4 skill | case-execution 不被安装，但既有 4 skill 不受影响 |
| import op_mapper 路径错误（Gotcha #2） | 修正 sys.path 注入逻辑；不回滚整文件 | case_runner 启动失败，既有 op_mapper 不受影响 |

### 8.2 feature flag（无需）

ST-EX-04 是新建 skill，不修改既有功能。case-execution skill 未安装时不影响 ptm-te 既有工作流（op_mapper/device-management/trex-traffic 既有 skill 独立可用）。无需 feature flag，通过 install.py 安装/卸载控制启用/禁用。

### 8.3 exec_v4.py 废弃策略（ADR-03，ST-EX-13 承载）

ST-EX-04 不删除 exec_v4.py（ADR-03）。exec_v4.py 废弃标记由 ST-EX-13 承载（24 用例整改时）。ST-EX-04 实现的 case_runner.py 作为替代入口，但 exec_v4.py 保留可回退。

## 9. 平台差异

### 9.1 canonical 与 workspace 路径映射

| 组件 | canonical 源（ptm-team） | workspace 安装目标（ptm-te） | 路径转换 |
|---|---|---|---|
| case_runner.py | skills/case-execution/scripts/case_runner.py | .claude/skills/case-execution/scripts/case_runner.py | install.py 安装时复制 |
| SKILL.md | skills/case-execution/SKILL.md | .claude/skills/case-execution/SKILL.md | install.py 安装时复制 |
| op_mapper.py | skills/policy-route-execution/scripts/op_mapper.py | .claude/skills/policy-route-execution/scripts/op_mapper.py | install.py 既有安装 |
| install.py | script/ptm_team/install.py | 不安装到 workspace | canonical 侧执行 |

### 9.2 跨 skill import 路径一致性（Gotcha #2）

case_runner.py 的 sys.path 注入使用相对路径 `../../policy-route-execution/scripts`：
- canonical 源：`skills/case-execution/scripts/` -> `../../policy-route-execution/scripts/` = `skills/policy-route-execution/scripts/` ✓
- workspace 安装：`.claude/skills/case-execution/scripts/` -> `../../policy-route-execution/scripts/` = `.claude/skills/policy-route-execution/scripts/` ✓

两个场景下相对路径均有效（case-execution 和 policy-route-execution 在同一父目录下）。

### 9.3 重装一致性（Gotcha #10，R-C-007）

install.py 安装 case-execution skill 后，重装（uninstall -> install）不丢失。验证方式：
1. `install.py install ptm-te --component full` -> 检查 `.claude/skills/case-execution/` 存在
2. `install.py uninstall ptm-te` -> 检查 `.claude/skills/case-execution/` 移除
3. `install.py install ptm-te --component full` -> 检查 `.claude/skills/case-execution/` 恢复
4. `check_installed_drift` -> case-execution skill 无漂移

PTM_TE_SKILLS 新增 case-execution 后，get_agent_skills("ptm-te") 返回 5 个 skill（含 case-execution），check_installed_drift 会校验 case-execution 的 source_hash/installed_hash 一致性。

## 10. 数据模型

### 10.1 用例文件结构（frontmatter + case_steps）

```markdown
---
用例编号: PC-M4-01-01                    # 必填，唯一标识（ADR-07）
用例名称: 创建策略路由-有效参数           # 必填
标签: [IPv4, 策略路由, P0]               # 可选，ST-EX-16 tag 过滤
关键词: 策略路由/有效参数                 # 可选，ST-EX-16 keyword 过滤
用例类型: 功能测试                        # 可选（frontmatter 16 列之一）
前置条件: 已登录Web管理                   # 可选（冗余列，AGA-03=C 忽略）
测试步骤: STEP-001..STEP-007             # 可选（冗余列，与 case_steps 冗余，忽略）
预期结果: 策略路由创建成功                 # 可选（冗余列，与 case_steps 冗余，忽略）
... (frontmatter 16 列，ST-EX-15 定义完整列名)
---

## case_steps

```yaml
- step_id: STEP-001
  step_name: 登录Web管理
  op_id: fw_login_web_management
  args:
    username: "admin"
  target: dut
- step_id: STEP-002
  step_name: 配置策略路由
  op_id: fw_config_policy_route
  args:
    source_network: "OBJ-SRC-192"
    dst_network: "OBJ-DST-10"
    next_hop_ip: "${ENV.dut.next_hop}"    # ${ENV.*} 占位符（ADR-09）
    in_interface: "any"
    type: "ipv4"
  target: dut
- step_id: STEP-003
  step_name: TG 配置接口
  op_id: tg_config_interface
  args:
    interfaces: ${ENV.tg.ports[port1,port2]}   # ${ENV.*} 聚合数组
  target: tg
```
```

**frontmatter 16 列处理**（AGA-03=C，Gotcha #7）：
- ST-EX-04 只消费 4 列：用例编号/用例名称/标签/关键词
- 其余 12 列保留在 frontmatter dict 供审计，但不参与执行逻辑
- 测试步骤/预期结果列与 case_steps YAML 冗余，case_runner 忽略（不校验内容一致性）
- 完整 16 列列名约定由 ST-EX-15 承载

### 10.2 env_topology 契约（HLD §12.3，ADR-09）

```yaml
# env-file / topology yaml 结构
port_mapping:
  port1:
    tg: "1/1/1"      # 必须加引号（YAML 1.1 整数陷阱，Gotcha #11）
    dut: "eth0"
  port2:
    tg: "1/1/2"
    dut: "eth1"
nodes:
  tg1:
    trex_api_url: "http://10.113.52.253:8450"   # TREX_API_URL 来源（ADR-05 环境文件优先）
    interfaces:
      port1:
        ip: "192.168.101.1"
        gw: "192.168.101.2"
      port2:
        ip: "192.168.102.1"
        gw: "192.168.102.2"
  dut1:
    host: "https://10.113.55.51"                # DUT URL（${ENV.dut.url}）
    next_hop: "192.168.102.1"                   # DUT 下一跳（${ENV.dut.next_hop}，ADR-09）
    interfaces:
      port1:
        ip: "192.168.101.2"
links:
  - from: tg1.port1
    to: dut1.port1
    name: link1
  - from: tg1.port2
    to: dut1.port2
    name: link2
```

### 10.3 result.json 结构（ST-EX-04 写入骨架，ST-EX-11 完善诊断）

```json
{
  "run_id": "run-20260728-160000",
  "mode": "dry-run",
  "started_at": "2026-07-28T16:00:00+08:00",
  "finished_at": "2026-07-28T16:04:30+08:00",
  "devices": {
    "dut_url": "https://10.113.55.51",
    "tg_api_server_source": "env_file"   // "env_file" | "devices_yaml" | "none"
  },
  "env_file": "configs/topology.yaml",
  "cases": [
    {
      "case_id": "PC-M4-01-01",
      "case_name": "创建策略路由-有效参数",
      "source_path": "cases/IPv4策略路由/配置管理/策略配置/PC-M4-01-01-创建策略路由-有效参数.md",
      "overall_status": "PASS",          // PASS/FAIL/KNOWN_FAIL/ERROR
      "steps": [
        {
          "step_id": "STEP-001",
          "step_name": "登录Web管理",
          "op_id": "fw_login_web_management",
          "status": "skipped",            // skipped（复用 SHARED_SESSION，Gotcha #8）
          "envelope": null
        },
        {
          "step_id": "STEP-002",
          "step_name": "配置策略路由",
          "op_id": "fw_config_policy_route",
          "status": "success",
          "envelope": {"op_id": "fw_config_policy_route", "status": "success", "data": {"policy_route_id": 42}}
        }
      ],
      "cleanup": {
        "status": "success",
        "ops_reversed": 3
      }
    }
  ],
  "summary": {
    "total": 1,
    "pass": 1,
    "fail": 0,
    "known_fail": 0,
    "error": 0
  },
  "logout_status": "n/a",               // dry-run 模式 n/a；--execute 模式 success/fallback
  "runtime_authorization": null          // dry-run 模式 null；--execute 模式审计字段（ST-EX-05）
}
```

### 10.4 report.md 结构（ST-EX-11 承载，ST-EX-04 预留调用点）

ST-EX-04 的 main 在 [8] 调用 generate_report(results)，report.md 模板由 ST-EX-11 设计。ST-EX-04 只保证 results 数据结构完整（§10.3），不实现 report.md 渲染。

## 11. 性能

### 11.1 性能指标（HLD §13.1）

| 指标 | 目标值 | 验证方式 | 说明 |
|---|---|---|---|
| 用例执行时间 | 24 用例 < 5 分钟（含 op 间 2s 间隔） | --execute 集成测试 | static-only 不可验证（DQ-01） |
| import 开销 | 无进程启动开销（ADR-01 import 模式） | 代码审查 | case_runner import op_mapper 函数调用，无 subprocess 启动新 Python |
| op 间间隔 | 2s | 代码审查 | time.sleep(2) |
| 用例间间隔 | 8s | 代码审查 | time.sleep(8) |
| TG op timeout | 90s（Gotcha #9） | 代码审查 | step.target=="tg" 时 timeout=90 |
| DUT op timeout | 45s（Gotcha #9） | 代码审查 | step.target=="dut" 时 timeout=45 |
| ConnectTimeout 重试 | TG 3 次（15/20/25s）/ DUT 1 次（30s） | 代码审查 | ST-EX-05 编排 |

### 11.2 性能保障措施

- **import 模式**（ADR-01）：case_runner 直接 import op_mapper，无 subprocess 启动开销（对比 exec_v4.py 方案 B 的 170 次进程启动）
- **共享 session**（Gotcha #8）：预登录建立 SHARED_SESSION，跨用例复用，避免每用例重复登录
- **timeout 分层**（Gotcha #9）：TG op 90s（含 ARP 学习），DUT op 45s，避免 TG 操作被 DUT timeout 截断

## 12. 安全

### 12.1 安全约束（HLD §13.4，ADR-02）

| 约束 | 实现 | 验证 |
|---|---|---|
| NO_CREDENTIAL_READ | resolve_addresses 不读取密码值，只返回环境变量名；密码经 --password-env 传环境变量名 | 代码审查：resolve_addresses 无密码字段读取 |
| NO_PRODUCTION_WRITE | dry-run 默认门（ADR-02）；--execute 需 --authorized 授权 | 代码审查：main 默认 dry_run=True |
| NO_EXTERNAL_PUBLISH | case_runner 不发布到外部；result.json/report.md 写入本地 runs/ 目录 | 代码审查：无网络发布代码 |
| dry-run 默认门 | argparse 默认 dry_run=True；--execute 需显式指定且需 --authorized | UT-EX-04-21 |
| runtime_authorization 审计 | --execute 模式记录 who/scope/authorized_at/reason（ST-EX-05 实现） | ST-EX-05 承载 |
| session 路径 | session_file 写入 ~/.local/state/ptm-atomic/，不写入仓库目录 | 代码审查 |
| 凭据占位 | devices.yaml 凭据用 ${ENV_VAR} 占位，禁止明文入库 | resolve_addresses 校验 |

### 12.2 ptm-atomic 约束（Gotcha #13，ADR-05/09）

- TG 操作必须经 `ptm-atomic run tg trex <action>` 原子操作下发
- 框架禁止直接调 TG REST API（无 requests/urllib 直接调 TG 的代码）
- TREX_API_URL 经 _build_exec_env 注入子进程环境变量，由 ptm-atomic 子进程消费
- resolve_env_refs 只解析参数，不发起网络请求

**code review 检查项**：case_runner.py 无 `import requests`/`import urllib` 直接调 TG 的代码；TG 操作全部经 op_mapper.execute_op -> build_command -> subprocess(ptm-atomic)。

### 12.3 设计通过不等于运行授权

CP5 设计确认通过不授权 --execute 运行。--execute 模式需独立 runtime_authorization 决策项（who/scope/authorized_at/reason），由 ST-EX-05 承载。ST-EX-04 的 dry-run 默认门保证设计通过后不会意外触发真实设备写操作。

## 13. 依赖

### 13.1 Story 依赖（DEVELOPMENT-PLAN.yaml）

| 依赖 Story | 依赖类型 | 依赖内容 | 影响 |
|---|---|---|---|
| ST-EX-03 | hard | op_mapper.execute_op 签名扩展（env_topology + tg_api_server 参数）；_build_exec_env 签名扩展；resolve_env_refs 实现 | ST-EX-04 消费扩展后签名；ST-EX-03 未完成时 ST-EX-04 无法调用带 env_topology 的 execute_op |

**ST-EX-03 未完成时的降级**：若 ST-EX-03 尚未完成 execute_op 签名扩展，ST-EX-04 可先实现不传 env_topology/tg_api_server 的版本（向后兼容，无 ${ENV.*} 用例原样透传，Gotcha #12），但 CP6 实现时必须等 ST-EX-03 完成后补全参数传递。file_ownership 无冲突（ST-EX-03 拥有 op_mapper.py，ST-EX-04 拥有 case_runner.py）。

### 13.2 外部依赖

| 依赖 | 类型 | 版本/约束 | 验证方式 |
|---|---|---|---|
| ptm-atomic CLI | 外部工具 | commit 952a625，version 0.1.0（install.py 规则块第 8 条） | which ptm-atomic（DA-001） |
| op_mapper.py | 同仓库 skill | 既有 21 op + ST-EX-03 扩展 | import op_mapper（Gotcha #2） |
| devices.yaml | 数据文件 | 含 firewall.host + tg.api_server（DA-003） | resolve_addresses 校验 |
| topology yaml / env-file | 数据文件 | 含 port_mapping/nodes/links（DA-006/007） | load_env_file 校验 |
| Python | 运行时 | 3.11（pyproject.toml） | uv run python |
| PyYAML | 依赖 | 既有（op_mapper 已依赖） | import yaml |

### 13.3 下游 Story 依赖 ST-EX-04

| 下游 Story | 依赖 ST-EX-04 内容 |
|---|---|
| ST-EX-05 | main 编排骨架 + execute_steps 调用点 |
| ST-EX-06 | main 用例后清理调用点 + step_refs_dir 路径约定 |
| ST-EX-07 | main finally 块 cleanup_session 调用点 |
| ST-EX-08 | main step 执行循环 apply_warming_up 调用点 |
| ST-EX-09 | main step 执行循环 apply_retry 调用点 |
| ST-EX-10 | main step 执行循环 classify_result 调用点 |
| ST-EX-11 | main [8] generate_report 调用点 + result.json 数据结构 |
| ST-EX-17 | main [2] build_env_topology/preconfigure_dut_interfaces 调用点 + env_topology 传入 |
| ST-EX-13 | case_runner.py 三入口 + parse_case_file 契约（24 用例整改后 dry-run 校验） |

## 14. 开放项

### 14.1 LLD Clarification Queue

| 问题 ID | 问题 | 推荐方案 | 备选 | blocks_lld | 状态 |
|---|---|---|---|---|---|
| LCQ-ST-EX-04-01 | --env-file 与 --topology-yaml 的关系：--env-file 缺省时是否复用 --topology-yaml 作为 env_topology 源？HLD §12.3 说"暂用 topology yaml 作环境文件载体"，但 §12.1 命令行示例两参数同时出现 | A：--env-file 缺省时复用 --topology-yaml（若含 port_mapping/nodes/links）作为 env_topology 源；--env-file 显式指定时优先用 --env-file | B：--env-file 和 --topology-yaml 完全独立；--env-file 缺省时不加载 env_topology（用例不可用 ${ENV.*}） | false | OPEN（已写入 process/state/QUESTION-LEDGER.ndjson；LLD 采用方案 A，非阻塞） |

### 14.2 遗留 OPEN 项（来自 HLD §20，ST-EX-04 消费）

| 问题 ID | 问题 | 状态 | ST-EX-04 影响 |
|---|---|---|---|
| O-02 | fw_logout op 在 ptm-atomic 安装版是否暴露（ADR-04） | OPEN（安装前验证） | ST-EX-04 main [6] 预留 cleanup_session 调用点；fw_logout 暴露验证由 ST-EX-07 承载，降级逻辑由 ST-EX-07 实现 |
| O-03 | 24 用例 known_issue 标注完整性（DQ-03） | OPEN（整改时检查） | ST-EX-04 parse_case_file 解析 known_issue 字段；标注完整性由 ST-EX-13 整改时检查 |
| O-04 | 环境文件 schema 管理 skill/CLI（后续 CR 候选） | OPEN（不阻塞 LLD） | ST-EX-04 load_env_file 只读 topology yaml 作载体；schema 管理 skill/CLI 属后续 CR |

### 14.3 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | ST-EX-04 LLD 对应章节 | 一致性 |
|---|---|---|
| HLD §12.1 case_runner 执行流程（七步） | §4.1 核心流程（七步 + 清理顺序 + dry-run 行为） | ✓ 一致 |
| HLD §12.2 _build_exec_env + execute_op 流程 | §3.6 消费契约（ST-EX-04 传 tg_api_server，ST-EX-03 实现注入） | ✓ 一致（边界显式划分） |
| HLD §12.3 环境解析层设计 | §3.5 load_env_file + §3.6 _resolve_tg_api_server + §4.2 清理顺序 | ✓ 一致（build_env_topology/preconfigure_dut_interfaces 归 ST-EX-17） |
| HLD §13.1 性能 | §11 性能 | ✓ 一致 |
| HLD §13.4 安全 | §12 安全 | ✓ 一致 |
| HLD §14 RA-013/RA-014 | §5.1 EnvFileError + §3.5 load_env_file 校验 | ✓ 一致 |
| ADR-01 import 集成 | §3.6 消费契约 + §5.3 import 路径处理 | ✓ 一致 |
| ADR-02 dry-run 默认门 | §4.3 dry-run 模式行为 + §12.1 安全约束 | ✓ 一致 |
| ADR-05 TREX_API_URL 注入（方案 a，环境文件优先） | §3.6 _resolve_tg_api_server（env_topology ${ENV.tg.url} 优先，devices.yaml fallback） | ✓ 一致（ST-EX-04 消费，ST-EX-03 实现注入） |
| ADR-08 TG 数据归属 | §3.4 resolve_addresses（devices.yaml tg.api_server fallback） | ✓ 一致 |
| ADR-09 环境文件驱动 | §3.5 load_env_file + §3.6 env_topology 传参 | ✓ 一致 |
| Gotcha #2 import 路径 | §5.3 import op_mapper 路径处理 | ✓ 一致 |
| Gotcha #7 frontmatter 冗余列 | §3.3 parse_case_file（AGA-03=C 忽略冗余列） | ✓ 一致 |
| Gotcha #8 共享 session | §4.3 dry-run 行为 + §4.1 [4] 预登录 | ✓ 一致 |
| Gotcha #9 timeout 分层 | §3.6 timeout 设置 + §11.1 性能 | ✓ 一致 |
| Gotcha #10 重装一致性 | §9.3 重装一致性 + §6.4 安装验证 | ✓ 一致 |
| Gotcha #11 YAML 1.1 整数陷阱 | §3.5 load_env_file 端口名类型校验 | ✓ 一致（完整校验归 ST-EX-17） |
| Gotcha #12 向后兼容 | §4.3 dry-run resolve_env_refs 执行 + §3.5 load_env_file None 返回 | ✓ 一致 |
| Gotcha #13 ptm-atomic 约束 | §12.2 ptm-atomic 约束 + §3.6 消费契约 | ✓ 一致 |

### 14.4 设计自审

- **内部一致性**：§3 接口设计与 §4 核心流程一致；§3.6 消费契约与 §3.7 下游边界一致；§10 数据模型与 §4 流程一致
- **目标量化**：§6 测试设计 23 单元测试 + 4 dry-run + 4 安装验证，全部含可度量预期
- **集成契约显式化**：§3.6 消费契约（调用方向/时机/输入/输出/后续衔接/降级）+ §3.7 下游边界表
- **前置校验与失败路径**：§5.1 异常分类 + §4.1 [1] 启动校验
- **回退决策可操作化**：§8 回滚策略 + §13.1 ST-EX-03 未完成降级
- **遗留问题状态闭环**：§14.1 LCQ + §14.2 OPEN 项全部有状态
- **Gotchas 覆盖**：Gotcha #2/#7/#8/#9/#10/#11/#12/#13 全部在对应章节落实
