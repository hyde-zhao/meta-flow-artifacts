---
cr_id: "CR-033"
story_id: "STORY-EX-15"
title: "用例结构化约定（目录结构 + 命名 + frontmatter 16 列约定）"
wave: 3
priority: "P1"
lld_policy: "technical-note"
feature_design_refs:
  - "docs/features/case-execution/DESIGN.md"
depends_on: ["ST-EX-04"]
dependency_type: "soft"
status: "lld-ready"
design_evidence_type: "technical-note"
lld_policy_required_level: "technical-note"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-29T16:00:00+08:00"
author: "meta-dev"
---

# ST-EX-15 用例结构化约定（目录结构 + 命名 + frontmatter 16 列约定）

## 设计证据类型

technical-note（FE-EX-02 required，Feature 级 DESIGN.md 承载 frontmatter 16 列解析逻辑/必填校验规则；本 Story 内技术说明承载目录/命名/frontmatter 约定 + parse_frontmatter 设计证据）。trigger_reasons：目录结构 + 命名 + frontmatter 16 列约定；rationale：约定文档，Story 内技术说明足够。

## 设计依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §12.1 [3]（用例发现 --cases-dir glob）+ [5]（解析 frontmatter 16 列，忽略测试步骤/预期结果列） | parse_frontmatter 在执行流程中的位置 |
| HLD | §12.1 [3]（--tag 按 tags 列精确过滤，--keyword 按关键词列模糊匹配） | tags/关键词列数据源 |
| HLD | §18 下沉（frontmatter 16 列解析逻辑/必填校验规则 -> FE-EX-02 DESIGN.md） | 解析逻辑由 Feature DESIGN.md 承载 |
| HLD | §4 灰区 AGA-03=C（frontmatter 冗余字段，摘要 + case_runner 忽略冗余列） | 测试步骤/预期结果列忽略 |
| HLD | ADR-07（用例命名与编号解析，编号正则匹配） | 命名约定 |
| HLD | Gotcha #6（用例名称连字符与文件名分隔符冲突）+ Gotcha #7（frontmatter 测试步骤/预期结果与 case_steps YAML 冗余） | 解析规避 |
| 需求 | R-F-022（目录结构 cases/三级/四级/五级/） | 三级目录结构 |
| 需求 | R-F-023（命名 <编号>-<名称>.md，编号保留组网前缀） | 命名约定 |
| 需求 | R-F-024（frontmatter 16 列，保留 case_steps YAML） | 16 列字段 |
| 需求 | R-F-025（tags 列结构化列表 + 关键词列逗号分隔） | tags/关键词列 |
| 决策 | DQ-05（8 必填+8 可选，缺失列填 N/A） | 必填校验 |
| 决策 | DQ-06（命名连字符冲突，以 frontmatter 用例编号列为唯一标识） | 编号正则 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 3 ST-EX-15 | file_ownership=SKILL.md#用例结构化约定 + case_runner.py#parse_frontmatter，depends_on=ST-EX-04 soft |

## 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `skills/case-execution/SKILL.md` | 用例结构化约定 | 新增章节 | 目录结构 + 命名 + frontmatter 16 列 + tags 列约定 |
| `skills/case-execution/scripts/case_runner.py` | `parse_frontmatter` | 新增函数 | 解析 frontmatter 16 列 + tags 列，忽略冗余列，必填校验 |
| `skills/case-execution/scripts/case_runner.py` | `_CASE_ID_RE` | 新增模块常量 | 用例编号正则 `^PC-[A-Z0-9]+-\d+-\d+-\d+` |
| `docs/features/case-execution/DESIGN.md` | frontmatter 16 列解析逻辑/必填校验 | 补充章节 | 由 ST-EX-04 主导，本 Story 补约定段 |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `skills/case-execution/SKILL.md#用例结构化约定`
- `skills/case-execution/scripts/case_runner.py#parse_frontmatter`

**不触碰文件**：`ptm-te/cases/**/*.md`（24 用例整改由 ST-EX-13 拥有，本 Story 只定义约定 + 解析函数）；`skills/policy-route-execution/scripts/op_mapper.py`（ST-EX-03/07 拥有）。

## 接口设计要点

### 目录结构约定（R-F-022）

```
cases/
├── IPv4策略路由/                    # 三级目录 = 功能模块
│   └── 配置管理/                    # 四级目录 = 子模块
│       └── 策略配置/                # 五级目录 = 用例组
│           ├── PC-COMB-M4-01-01-五元组匹配-源IP目的IP全匹配命中验证.md
│           └── PC-COMB-M4-01-03-策略ID不存在-创建失败.md
└── IPv6策略路由/                    # 三级目录（未来扩展）
```

**层级语义**：
- 三级 = 功能模块（如 `IPv4策略路由`），对应 frontmatter `三级目录` 列
- 四级 = 子模块（如 `配置管理`），对应 frontmatter `四级目录` 列
- 五级 = 用例组（如 `策略配置`），对应 frontmatter `五级目录` 列
- 现有 `cases/upload/` 24 用例需迁移到新结构（ST-EX-13 落地）；旧 `cases/upload/` 保留废弃标记不含 .md（Gotcha #5）

**case_runner 消费**：`--cases-dir cases/IPv4策略路由/` 递归 glob 扫描 `.md` 文件；目录路径不作为执行依据（执行依据是 frontmatter 用例编号列），目录仅用于组织。

### 命名约定（R-F-023, ADR-07, Gotcha #6）

**文件名格式**：`<用例编号>-<用例名称>.md`

- 用例编号：保留组网前缀，格式 `PC-<组网>-<模块>-<序号>-<子序号>`，如 `PC-COMB-M4-01-01`
- 用例名称：来自 frontmatter `用例名称` 列，名称内部连字符保留
- 编号与名称之间用**单个连字符**分隔
- 示例：`PC-COMB-M4-01-01-五元组匹配-源IP目的IP全匹配命中验证.md`

**连字符冲突规避（DQ-06, Gotcha #6）**：文件名含多个连字符（编号含 `-`，名称含 `-`），直接 `split('-')` 会解析错误。parse_frontmatter 以 frontmatter `用例编号` 列为唯一标识，文件名解析用正则匹配编号前缀：

```python
_CASE_ID_RE = re.compile(r"^(PC-[A-Z0-9]+-\d+-\d+-\d+)")

def _parse_case_id_from_filename(filename: str) -> tuple:
    """从文件名解析用例编号和名称。

    Returns:
        (case_id, case_name) 元组；编号不匹配时 case_id=None
    """
    stem = filename.rsplit(".", 1)[0]  # 去扩展名
    m = _CASE_ID_RE.match(stem)
    if not m:
        return (None, stem)
    case_id = m.group(1)
    case_name = stem[len(case_id)+1:]  # 去编号 + 分隔连字符
    return (case_id, case_name)
```

**一致性校验**：parse_frontmatter 解析后，校验文件名解析的 case_id 与 frontmatter `用例编号` 列一致；不一致时记录 warning（不阻塞执行，以 frontmatter 为准）。

### frontmatter 16 列 + tags 列约定（R-F-024, R-F-025）

**frontmatter 格式**：YAML frontmatter（`---` 分隔），位于 md 文件顶部，case_steps YAML 块之前：

```yaml
---
三级目录: IPv4策略路由
四级目录: 配置管理
五级目录: 策略配置
用例名称: 五元组匹配-源IP+目的IP全匹配命中验证
用例编号: PC-COMB-M4-01-01
用例级别: P0
组网描述: TOPO-01 基础三链路
组网约束: node2_dut1_tg1_link3
预置条件: DUT 已启动，TG 已连接
测试步骤: STEP-001..STEP-012
预期结果: 策略路由命中，命中计数 > 0
首次创建版本: v1.0
最后变更版本: v1.0
关键词: fw_config_policy_route, fw_verify_policy_route, S01
测试类型: 正向测试
是否自动化: "true"
tags: [策略路由, IPv4, PC, 正向, P0]
---

## case_steps

```yaml
case_steps:
- step_id: STEP-001
  ...
```
```

**16 列清单**（R-F-024，顺序固定）：
1. 三级目录 2. 四级目录 3. 五级目录 4. 用例名称 5. 用例编号 6. 用例级别 7. 组网描述 8. 组网约束 9. 预置条件 10. 测试步骤 11. 预期结果 12. 首次创建版本 13. 最后变更版本 14. 关键词 15. 测试类型 16. 是否自动化

**tags 列**（R-F-025，第 17 字段，附加于 16 列之外）：结构化列表 `[策略路由, IPv4, PC, 正向, P0]`，供 `--tag` 精确过滤。

**关键词列**（16 列之第 14 列）：逗号分隔 `fw_config_policy_route, fw_verify_policy_route, S01`，供 `--keyword` 模糊匹配。

**必填/可选（DQ-05，LCQ-ST-EX-15-01 推荐方案 A）**：

| 必填（8） | 可选（8） | 附加 |
|---|---|---|
| 用例名称 / 用例编号 / 用例级别 / 组网描述 / 预置条件 / 测试步骤 / 预期结果 / 测试类型 | 三级目录 / 四级目录 / 五级目录 / 组网约束 / 首次创建版本 / 最后变更版本 / 关键词 / 是否自动化 | tags（可选） |

- 必填缺失 -> parse_frontmatter 记录 error，该用例不执行（标记 INVALID）
- 可选缺失 -> 填 `N/A`（RA-012），继续执行
- tags 缺失 -> 填空列表 `[]`，`--tag` 过滤时不匹配该用例
- 测试步骤/预期结果虽冗余（AGA-03=C），但保留作摘要**必填**（DQ-05 不删列）；parse_frontmatter 校验存在性，**忽略内容**（case_steps YAML 是真相源）

**冗余列忽略（AGA-03=C, Gotcha #7）**：parse_frontmatter 解析"测试步骤"和"预期结果"两列时只校验字段存在，不读取内容；step 执行数据全部来自 case_steps YAML 块。校验脚本检查 16 列存在性，不校验内容一致性。

### parse_frontmatter 签名

```python
_REQUIRED_FIELDS = ["用例名称", "用例编号", "用例级别", "组网描述",
                    "预置条件", "测试步骤", "预期结果", "测试类型"]
_OPTIONAL_FIELDS = ["三级目录", "四级目录", "五级目录", "组网约束",
                    "首次创建版本", "最后变更版本", "关键词", "是否自动化"]
_ALL_16_FIELDS = _REQUIRED_FIELDS + _OPTIONAL_FIELDS  # 16 列

def parse_frontmatter(md_text: str) -> dict:
    """解析用例 md 的 YAML frontmatter，校验 16 列 + tags 列。

    Args:
        md_text: 用例 md 文件全文

    Returns:
        解析结果 dict：
        {
            "fields": {字段名: 值, ...16 列...},  # 可选缺失填 N/A
            "tags": ["策略路由", ...],            # 结构化列表，缺失填 []
            "keywords": ["fw_config_policy_route", ...],  # 关键词列逗号分割
            "case_id": "PC-COMB-M4-01-01",        # 用例编号列
            "errors": ["必填字段'用例名称'缺失"],  # 校验错误列表
            "warnings": ["文件名编号与frontmatter不一致"]
        }
        - frontmatter 块缺失 -> errors 含 "frontmatter 缺失"，fields={}
        - 必填缺失 -> errors 记录缺失字段名
        - 测试步骤/预期结果 -> 只校验存在，内容不解析（忽略）

    消费方：
    - discover_cases: 按文件名 glob + parse_frontmatter 构建用例清单
    - filter_by_tag/filter_by_keyword (ST-EX-16): 消费 tags/keywords
    - execute_steps: 消费 case_id（step-refs 落盘命名）
    """
```

**关键词列解析**：`关键词` 列逗号分隔字符串，parse_frontmatter split 为列表；`tags` 列已是 YAML 列表，直接取。

**dry-run 行为**：dry-run 模式下 parse_frontmatter 仍执行（校验 frontmatter 完整性 + 必填校验），errors 用例标记 INVALID 不执行（dry-run 只校验不执行 op，但用例结构校验必须通过）。

## 实施步骤

1. **SKILL.md 新增"用例结构化约定"章节**：目录结构（三级/四级/五级）+ 命名（`<编号>-<名称>.md`）+ frontmatter 16 列清单 + tags 列 + 必填/可选划分 + 冗余列忽略说明 + case_steps YAML 块保留声明
2. **定义 `_CASE_ID_RE` / `_REQUIRED_FIELDS` / `_OPTIONAL_FIELDS` / `_ALL_16_FIELDS` 常量**：编号正则 + 字段清单
3. **实现 `parse_frontmatter`**：提取 `---` 分隔的 YAML frontmatter 块 -> yaml.safe_load 解析 -> 校验 16 列存在性（必填缺失记 error，可选缺失填 N/A）-> 解析 tags 列表 + 关键词列 split -> 返回结构化 dict
4. **实现 `_parse_case_id_from_filename`**：正则匹配编号前缀，返回 (case_id, case_name)；与 frontmatter 用例编号列一致性校验
5. **接入 discover_cases**：ST-EX-04 discover_cases glob 扫描 .md 后调 parse_frontmatter 构建用例清单（depends_on=ST-EX-04 soft，约定文档可先写，函数接入在 ST-EX-04 主干）
6. **必填校验与 INVALID 标记**：parse_frontmatter errors 非空的用例标记 INVALID，discover_cases 跳过执行并记入 result.json
7. **dry-run 校验**：dry-run 跑含 frontmatter 的用例，验证 16 列校验 + tags/关键词解析

## 回滚策略

- 本 Story 为 SKILL.md 新增章节 + case_runner.py 新增函数/常量，不修改 op_mapper，不改变 case_steps YAML 格式
- 回滚方式：git revert 相关 commit；移除 parse_frontmatter 后 discover_cases 回退到不解析 frontmatter（用例仍可按文件名执行，无 --tag/--keyword 过滤能力）
- 无运行时副作用：parse_frontmatter 是纯解析，不触发设备操作

## 权限与风险

- **权限约束**：遵循 NO_CREDENTIAL_READ / NO_PRODUCTION_WRITE / NO_EXTERNAL_PUBLISH；parse_frontmatter 是纯解析不触发设备操作
- **数据安全**：frontmatter 含用例元数据（编号/名称/tags）但不含凭据；用例 md 不入库 session
- **失败处理**：frontmatter 缺失/必填缺失记 error 标记 INVALID 不执行；编号不匹配记 warning 不阻塞
- **风险**：纯新增章节 + 函数，无运行时副作用；回滚 git revert

## 测试要点

| 测试项 | 验证方法 | 预期结果 |
|---|---|---|
| 16 列完整解析 | 单元测试：frontmatter 含全 16 列 + tags | fields 16 列 + tags 列表 |
| 必填缺失校验 | 单元测试：缺"用例名称" | errors 含"用例名称"缺失 |
| 可选缺失填 N/A | 单元测试：缺"首次创建版本" | fields["首次创建版本"]="N/A" |
| tags 列表解析 | 单元测试：tags=[策略路由,IPv4] | tags=["策略路由","IPv4"] |
| tags 缺失 | 单元测试：无 tags 列 | tags=[] |
| 关键词列 split | 单元测试：关键词="a, b, c" | keywords=["a","b","c"] |
| 冗余列忽略 | 单元测试：测试步骤="STEP-001..N" | fields 含键但内容不解析（case_steps 为准） |
| 编号正则匹配 | 单元测试：文件名 PC-COMB-M4-01-01-名称.md | case_id="PC-COMB-M4-01-01" |
| 编号不匹配 | 单元测试：文件名 无前缀.md | case_id=None |
| 文件名与 frontmatter 一致性 | 单元测试：文件名编号 != frontmatter 编号 | warning 记录 |
| frontmatter 缺失 | 单元测试：md 无 --- 块 | errors 含"frontmatter 缺失" |
| case_steps YAML 保留 | 单元测试：frontmatter 后有 case_steps 块 | case_steps 不被 parse_frontmatter 解析（由 ST-EX-04 parse_case_file 处理） |
| dry-run 校验 | dry-run 跑用例 | 16 列校验 + tags 解析执行 |
| 必填划分（LCQ-ST-EX-15-01） | 审查 _REQUIRED_FIELDS | 8 必填字段（待 host-orchestrator 确认） |

## 开放项

| 问题 ID | 问题 | 状态 | 说明 |
|---|---|---|---|
| LCQ-ST-EX-15-01 | DQ-05 的 8 必填/8 可选具体划分 + tags 列是否算 16 列 | OPEN（blocks_lld=false） | 已写入 QUESTION-LEDGER，推荐方案 A（必填=名称/编号/级别/组网/预置/步骤/预期/类型；tags 第 17 字段可选）。本 Story 按推荐方案实现，待 host-orchestrator 确认 |
| O-EX-15-01 | frontmatter 字段名是否支持英文别名 | OPEN（非阻塞） | 当前用中文字段名（与 R-F-024 一致）。若用例作者偏好英文键名，后续可加别名映射。CR-033 范围内按中文字段名 |

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 |
|---|---|
| HLD §12.1 [3]（--cases-dir glob + --tag/--keyword） | 目录结构 + tags/关键词列数据源 |
| HLD §12.1 [5]（解析 frontmatter 16 列，忽略测试步骤/预期结果列） | parse_frontmatter 冗余列忽略 |
| HLD §18 下沉（frontmatter 16 列解析/必填校验 -> FE-EX-02 DESIGN.md） | 约定补入 DESIGN.md |
| HLD AGA-03=C（冗余字段，忽略冗余列） | 测试步骤/预期结果只校验存在忽略内容 |
| ADR-07（命名与编号解析，正则匹配） | _CASE_ID_RE + _parse_case_id_from_filename |
| Gotcha #6（连字符冲突） | 编号正则匹配，frontmatter 编号为唯一标识 |
| Gotcha #7（frontmatter 与 case_steps 冗余） | 忽略冗余列，case_steps 是真相源 |
| R-F-022（目录三级结构） | cases/三级/四级/五级/ 约定 |
| R-F-023（命名 <编号>-<名称>.md） | 命名约定 + 编号正则 |
| R-F-024（frontmatter 16 列 + case_steps 保留） | 16 列清单 + parse_frontmatter |
| R-F-025（tags 列 + 关键词列） | tags 第 17 字段 + 关键词列 split |
| DQ-05（8 必填+8 可选） | _REQUIRED_FIELDS/_OPTIONAL_FIELDS（推荐方案 A） |
| DQ-06（命名冲突，编号为唯一标识） | _CASE_ID_RE 一致性校验 |
