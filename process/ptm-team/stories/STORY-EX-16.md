---
cr_id: "CR-033"
story_id: "STORY-EX-16"
title: "标签/关键字执行（--tag 精确过滤 + --keyword 模糊匹配）"
wave: 3
priority: "P1"
lld_policy: "technical-note"
feature_design_refs:
  - "docs/features/case-execution/DESIGN.md"
depends_on: ["ST-EX-04", "ST-EX-15"]
dependency_type: "hard"
status: "lld-ready"
design_evidence_type: "technical-note"
lld_policy_required_level: "technical-note"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-29T16:00:00+08:00"
author: "meta-dev"
---

# ST-EX-16 标签/关键字执行（--tag 精确过滤 + --keyword 模糊匹配）

## 设计证据类型

technical-note（FE-EX-02 required，Feature 级 DESIGN.md 承载用例发现逻辑；本 Story 内技术说明承载 --tag/--keyword 过滤逻辑的设计证据）。trigger_reasons：--tag 精确过滤 + --keyword 模糊匹配；rationale：逻辑简单，Story 内技术说明足够。

## 设计依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §12.1 [3]（--tag 按 frontmatter tags 列精确过滤；--keyword 按关键词列模糊匹配） | 过滤在用例发现阶段 |
| HLD | §10 模块表（case_runner.py 用例发现 --tag/--keyword） | 职责归属 case_runner |
| HLD | §18 下沉（frontmatter 16 列解析逻辑 -> FE-EX-02 DESIGN.md） | tags/关键词列由 ST-EX-15 parse_frontmatter 解析 |
| HLD | §8 场景 2（目录 glob 批量执行）+ UC-EX-02/03 | 三入口执行场景 |
| 需求 | R-F-004（三入口 --case-file/--cases-dir/--tag或--keyword） | --tag/--keyword 入口 |
| 需求 | R-F-025（tags 列结构化列表 + 关键词列逗号分隔） | 过滤数据源 |
| 需求 | R-F-026（--tag 精确过滤 AND/OR + --keyword 子串匹配 + 组合） | 过滤逻辑 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 3 ST-EX-16 | file_ownership=case_runner.py#filter_by_tag + #filter_by_keyword，depends_on=ST-EX-04 + ST-EX-15 hard |

## 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | `filter_by_tag` | 新增函数 | 按 frontmatter tags 列精确过滤（AND/OR） |
| `skills/case-execution/scripts/case_runner.py` | `filter_by_keyword` | 新增函数 | 按 frontmatter 关键词列模糊匹配（子串） |
| `skills/case-execution/scripts/case_runner.py` | argparse `--tag`/`--keyword`/`--tag-mode` | 新增 CLI 参数 | run 子命令入口 |
| `docs/features/case-execution/DESIGN.md` | --tag/--keyword 过滤逻辑 | 补充章节 | 由 ST-EX-04 主导，本 Story 补过滤段 |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `skills/case-execution/scripts/case_runner.py#filter_by_tag`
- `skills/case-execution/scripts/case_runner.py#filter_by_keyword`

**不触碰文件**：`skills/policy-route-execution/scripts/op_mapper.py`（过滤是 case_runner 编排层逻辑，op_mapper 不感知 tag/keyword）；`ptm-te/cases/**/*.md`（tags/关键词列标注由 ST-EX-13 整改，本 Story 只消费）。

## 接口设计要点

### 过滤数据源（ST-EX-15 parse_frontmatter 产出）

filter_by_tag / filter_by_keyword 消费 ST-EX-15 `parse_frontmatter` 的产出：

```python
{
    "fields": {...16 列...},
    "tags": ["策略路由", "IPv4", "PC", "正向", "P0"],   # 结构化列表
    "keywords": ["fw_config_policy_route", "fw_verify_policy_route", "S01"],  # 关键词列 split
    "case_id": "PC-COMB-M4-01-01",
    ...
}
```

- `tags`：frontmatter `tags` 列（YAML 列表），`--tag` 精确匹配此列表
- `keywords`：frontmatter `关键词` 列（逗号分隔 split 为列表），`--keyword` 子串匹配此列表

**依赖关系**（depends_on=ST-EX-15 hard）：filter_by_tag/filter_by_keyword 依赖 parse_frontmatter 已解析 tags/keywords 字段；ST-EX-15 未完成时无法过滤。

### --tag 精确过滤（R-F-026）

```python
def filter_by_tag(cases: list, tags: list, mode: str = "and") -> list:
    """按 frontmatter tags 列精确过滤。

    Args:
        cases: discover_cases + parse_frontmatter 构建的用例清单
        tags: 待过滤的 tag 列表（来自 --tag 逗号分隔）
        mode: "and"（默认，用例 tags 须全部含待过滤 tags）或
              "or"（用例 tags 含任一待过滤 tag）

    Returns:
        过滤后的用例子集

    匹配规则：
    - 精确匹配（大小写敏感）：待过滤 tag 须与用例 tags 列某元素完全相等
    - and 模式：用例 tags 须包含全部待过滤 tags（子集关系）
    - or 模式：用例 tags 须包含任一待过滤 tag（交集非空）
    - 用例 tags 为空列表 -> 不匹配任何 tag（and/or 均排除）
    - tags 参数为空 -> 不过滤，返回全部用例（filter_by_tag 透传）
    """
```

**AND/OR 示例（R-F-026）**：
- `--tag 策略路由,P0`（mode=and）-> 用例 tags 须同时含"策略路由"和"P0"
- `--tag 策略路由,P0 --tag-mode or` -> 用例 tags 含"策略路由"或"P0"任一
- `--tag 策略路由` -> 用例 tags 含"策略路由"（单 tag，and/or 等价）

### --keyword 模糊匹配（R-F-026）

```python
def filter_by_keyword(cases: list, keyword: str) -> list:
    """按 frontmatter 关键词列模糊匹配（子串匹配）。

    Args:
        cases: discover_cases + parse_frontmatter 构建的用例清单
        keyword: 待匹配关键字（来自 --keyword，单个字符串）

    Returns:
        过滤后的用例子集

    匹配规则：
    - 子串匹配（大小写敏感）：用例 keywords 列任一元素含 keyword 子串
    - 示例：--keyword fw_config -> 匹配含 "fw_config_policy_route" 的用例
    - 用例 keywords 为空 -> 不匹配
    - keyword 为空 -> 不过滤，返回全部用例
    """
```

**多关键字**：`--keyword` 支持 multiple（argparse `action="append"`），多个 --keyword 之间为 AND 关系（用例须同时满足所有关键字子串匹配）。

### --tag + --keyword 组合（R-F-026）

两者可组合使用，组合关系为 **AND**（同时满足 tag 过滤和 keyword 过滤）：

```python
# discover_cases 流程
cases = discover_cases(cases_dir)           # glob 扫描 + parse_frontmatter
if args.tag:
    cases = filter_by_tag(cases, args.tag.split(","), mode=args.tag_mode)
if args.keyword:
    for kw in args.keyword:                  # multiple --keyword AND
        cases = filter_by_keyword(cases, kw)
# cases 为最终执行用例集
```

**组合示例（R-F-026）**：
- `--tag 策略路由 --keyword S01` -> 用例 tags 含"策略路由" **且** keywords 列含"S01"子串

### CLI 参数设计

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `--tag` | string (逗号分隔) | None | 按 tags 列精确过滤，逗号分隔多 tag |
| `--tag-mode` | choice [and, or] | and | 多 tag 匹配模式（AND 全部满足 / OR 任一满足） |
| `--keyword` | string (append) | None | 按关键词列子串匹配，可多次指定（AND） |
| `--cases-dir` | path | - | 目录 glob 扫描（与 --tag/--keyword 组合） |
| `--case-file` | path | - | 单用例（--tag/--keyword 对单用例无意义，忽略并 warning） |

**--tag/--keyword 与入口的关系**：
- `--cases-dir` + `--tag`：先 glob 扫描目录全部 .md，再按 tag 过滤
- `--cases-dir` + `--keyword`：先 glob 扫描，再按 keyword 过滤
- `--cases-dir` + `--tag` + `--keyword`：先 glob，再 tag 过滤，再 keyword 过滤（AND）
- `--case-file` + `--tag`：单用例指定 --tag 无意义（无过滤对象），记录 warning 并忽略 --tag
- `--case-file` 单用：不过滤，直接执行该用例

### dry-run 行为

dry-run 模式下过滤逻辑仍执行（验证 --tag/--keyword 能正确匹配用例），匹配结果打印但不实际执行 op。目的：dry-run 校验过滤条件能匹配到预期用例集。

### 空结果处理

- 过滤后用例集为空 -> case_runner 打印 warning "过滤后无匹配用例"，result.json 记录 `filtered_count=0`，正常退出（exit code 0，非错误）
- 避免空结果误判为执行失败

## 实施步骤

1. **实现 `filter_by_tag`**：遍历 cases，按 mode（and/or）精确匹配 tags 列；空 tags 透传
2. **实现 `filter_by_keyword`**：遍历 cases，关键词列任一元素含 keyword 子串；空 keyword 透传
3. **argparse 新增 `--tag`/`--tag-mode`/`--keyword`**：run 子命令参数；`--keyword` 用 `action="append"` 支持多次
4. **接入 discover_cases 流程**：ST-EX-04 discover_cases glob + parse_frontmatter 后，按 args 调 filter_by_tag / filter_by_keyword 链式过滤（depends_on=ST-EX-04 + ST-EX-15 hard）
5. **--case-file + --tag warning**：单用例入口指定 --tag/--keyword 时记录 warning 并忽略
6. **空结果处理**：过滤后空集打印 warning + result.json 记录 filtered_count=0，exit 0
7. **dry-run 校验**：dry-run 跑 --tag/--keyword，打印匹配用例清单不执行 op

## 回滚策略

- 本 Story 为 case_runner.py 纯新增函数 + argparse 参数，不修改 op_mapper，不改变 frontmatter 格式
- 回滚方式：git revert 相关 commit；移除后 case_runner 回退到 --cases-dir 全量执行（无 tag/keyword 过滤），不影响 op 执行
- 无运行时副作用：过滤是纯计算（列表筛选），不触发设备操作

## 权限与风险

- **权限约束**：遵循 NO_CREDENTIAL_READ / NO_PRODUCTION_WRITE / NO_EXTERNAL_PUBLISH；filter_by_tag/keyword 是纯过滤不触发设备操作
- **数据安全**：消费 parse_frontmatter 的 tags/keywords 数据，不含凭据
- **失败处理**：过滤后空结果 warning + exit 0（非错误）；--case-file + --tag warning 忽略；不崩溃
- **风险**：纯新增函数 + argparse 参数，无运行时副作用；回滚 git revert

## 测试要点

| 测试项 | 验证方法 | 预期结果 |
|---|---|---|
| tag 精确匹配 | 单元测试：用例 tags=[策略路由,IPv4]，--tag 策略路由 | 匹配 |
| tag 大小写敏感 | 单元测试：--tag 策略路由 vs tags=[ipv4] | 不匹配（大小写敏感） |
| 多 tag AND | 单元测试：--tag 策略路由,P0 mode=and，tags=[策略路由,P0,正向] | 匹配（子集） |
| 多 tag AND 不全 | 单元测试：--tag 策略路由,P0 mode=and，tags=[策略路由,正向] | 不匹配（缺 P0） |
| 多 tag OR | 单元测试：--tag 策略路由,P0 mode=or，tags=[策略路由,正向] | 匹配（含策略路由） |
| 用例 tags 空 | 单元测试：tags=[]，--tag 策略路由 | 不匹配 |
| --tag 空 | 单元测试：--tag 未指定 | 透传全部用例 |
| keyword 子串匹配 | 单元测试：--keyword fw_config，keywords=[fw_config_policy_route] | 匹配 |
| keyword 不匹配 | 单元测试：--keyword xyz，keywords=[fw_config] | 不匹配 |
| 多 --keyword AND | 单元测试：--keyword fw_config --keyword S01 | 用例须同时含两子串 |
| --keyword 空 | 单元测试：--keyword 未指定 | 透传全部用例 |
| tag + keyword 组合 | 单元测试：--tag 策略路由 --keyword S01 | 同时满足（AND） |
| --cases-dir + --tag | 集成测试：目录 glob + tag 过滤 | 先 glob 再过滤 |
| --case-file + --tag warning | 集成测试：单用例 + --tag | warning 记录 + 忽略 --tag |
| 空结果 | 单元测试：--tag 不存在 | filtered_count=0, exit 0, warning |
| dry-run 过滤 | dry-run + --tag | 打印匹配清单不执行 op |
| 依赖 parse_frontmatter | 审查代码 | filter_by_tag 消费 parse_frontmatter 产出（ST-EX-15 依赖） |

## 开放项

| 问题 ID | 问题 | 状态 | 说明 |
|---|---|---|---|
| O-EX-16-01 | --tag/--keyword 是否支持正则匹配 | OPEN（非阻塞） | 当前 --tag 精确匹配、--keyword 子串匹配。正则匹配作为后续增强候选，CR-033 范围内 R-F-026 只要求精确+子串 |
| O-EX-16-02 | tags 列大小写敏感性 | OPEN（非阻塞） | 当前大小写敏感（"策略路由" != "策略路由"大小写差异）。中文无大小写问题，英文 tag 若有大小写差异需统一。ST-EX-13 整改时统一 tag 命名风格 |

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 |
|---|---|
| HLD §12.1 [3]（--tag 精确过滤 + --keyword 模糊匹配） | filter_by_tag + filter_by_keyword |
| HLD §10 模块表（case_runner 用例发现 --tag/--keyword） | 过滤归属 case_runner |
| HLD §8 场景 2（目录 glob 批量执行） | --cases-dir + --tag/--keyword 组合 |
| R-F-004（三入口 --case-file/--cases-dir/--tag或--keyword） | argparse --tag/--keyword 入口 |
| R-F-025（tags 列 + 关键词列数据源） | 消费 parse_frontmatter tags/keywords |
| R-F-026（--tag AND/OR + --keyword 子串 + 组合） | --tag-mode and/or + 子串匹配 + AND 组合 |
| DQ-CP3-03（tags 标注由 ST-EX-13 整改） | 本 Story 只消费 tags，标注由 ST-EX-13 |
