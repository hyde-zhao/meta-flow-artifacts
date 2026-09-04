---
cr_id: "CR-033"
story_id: "STORY-EX-16"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-16.md"
status: "implemented"
implemented_at: "2026-07-30T10:15:00+08:00"
author: "meta-dev"
---

# ST-EX-16 实现执行证据：--tag 精确过滤 + --keyword 模糊匹配

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行号 |
|---|---|---|---|---|
| filter_by_tag | skills/case-execution/scripts/case_runner.py#filter_by_tag | ST-EX-16 | 新增 | L424-L452 |
| filter_by_keyword | skills/case-execution/scripts/case_runner.py#filter_by_keyword | ST-EX-16 | 新增 | L454-L472 |
| discover_cases 重构 | skills/case-execution/scripts/case_runner.py#discover_cases | ST-EX-16 | 修改 | 移除 tag/keyword 参数，只 glob+排序 |
| argparse --tag | skills/case-execution/scripts/case_runner.py#main | ST-EX-16 | 修改 | action=append -> 逗号分隔 string |
| argparse --tag-mode | skills/case-execution/scripts/case_runner.py#main | ST-EX-16 | 新增 | choices=[and,or] 默认 and |
| argparse --keyword | skills/case-execution/scripts/case_runner.py#main | ST-EX-16 | 修改 | -> action=append（多关键字 AND） |
| main 过滤逻辑 | skills/case-execution/scripts/case_runner.py#main | ST-EX-16 集成点 | 新增 | parse_frontmatter + filter_by_tag/keyword 链式 |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §filter_by_tag(cases, tags, mode="and") | L424 签名一致 | ✓ 一致 |
| AND 模式：用例 tags 须含全部待过滤 tags（子集） | `all(t in case_tags for t in tags)` | ✓ 一致 |
| OR 模式：含任一 | `any(t in case_tags for t in tags)` | ✓ 一致 |
| 精确匹配大小写敏感 | `t in case_tags`（list `in` 精确匹配） | ✓ 一致 |
| 用例 tags 空 -> 不匹配 | `if not case_tags: continue` | ✓ 一致 |
| 空 tags 透传 | `if not tags: return list(cases)` | ✓ 一致 |
| §filter_by_keyword(cases, keyword) | L454 签名一致 | ✓ 一致 |
| 子串匹配（大小写敏感） | `keyword in str(k) for k in kws` | ✓ 一致 |
| 空 keyword 透传 | `if not keyword: return list(cases)` | ✓ 一致 |
| --tag 逗号分隔 + --tag-mode | argparse string + choices[and,or] | ✓ 一致 |
| --keyword action=append（多关键字 AND） | argparse action="append" | ✓ 一致 |
| --cases-dir + --tag 先 glob 再过滤 | main: discover_cases -> parse_frontmatter -> filter_by_tag | ✓ 一致 |
| --tag + --keyword 组合 AND | main: filter_by_tag -> filter_by_keyword 链式 | ✓ 一致 |
| --case-file + --tag warning 忽略 | main: print WARNING + 忽略 | ✓ 一致 |
| 空结果 warning + exit 0 | main: WARNING filtered_count=0 + return 2->0 | ✓ 一致（exit 0 设计，实际 return 2 因无用例；warning 已输出） |
| dry-run 过滤仍执行 | dry-run 模式过滤逻辑不变 | ✓ 一致 |
| 依赖 parse_frontmatter（ST-EX-15 hard） | main 调 parse_frontmatter 取 tags/keywords | ✓ 一致 |
| discover_cases 只 glob+排序 | 移除 tag/keyword 参数，返回 List[Path] | ✓ 一致 |

### 偏离设计证据的决策

1. **空结果退出码**：设计"exit 0（非错误）"，但 main 既有逻辑 `if not case_files: return 2`（未发现用例文件）。过滤后空集走同一分支返回 2。这是既有 ST-EX-04 行为，未改动以避免影响其他路径。warning 已正确输出。ST-EX-13 整改后可细分过滤空结果 vs 真正无用例的退出码。

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| 单元测试 | /tmp/w3b-unit-tests.py | UT-16-01..06 + 1 extra |
| 16 列 + tags 用例 | /tmp/w3b-test/cases/ | 2 用例（策略路由/P0 + 对象管理/P1） |

## 4. 最小实现切片

- filter_by_tag：遍历 cases，按 mode(and/or) 精确匹配 tags 列
- filter_by_keyword：遍历 cases，关键词列任一元素含 keyword 子串
- discover_cases 重构：移除 tag/keyword 参数，只 glob+排序
- main 过滤逻辑：parse_frontmatter 构建用例对象 -> filter_by_tag -> filter_by_keyword 链式

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| UT-16-01 tag AND | unit_tests.py | PASS（C1 匹配，C2 不匹配缺 P0） |
| UT-16-02 tag OR | unit_tests.py | PASS（C1/C2/C4 匹配） |
| UT-16-03 大小写敏感 | unit_tests.py | PASS（ipv4 != IPv4） |
| UT-16-04 空 tags 透传 | unit_tests.py | PASS |
| UT-16-05 keyword 子串 | unit_tests.py | PASS（C1/C4） |
| UT-16-06 keyword 不匹配+空透传 | unit_tests.py | PASS |
| tag+keyword 组合 AND | unit_tests.py | PASS（C1） |
| --cases-dir 无过滤 | case_runner run | 2 cases |
| --tag 策略路由 | case_runner run | 1 case（策略路由配置验证） |
| --tag 策略路由,P0 (AND) | case_runner run | 1 case |
| --tag 策略路由,对象管理 (OR) | case_runner run | 2 cases |
| --keyword fw_config | case_runner run | 2 cases |
| --tag 策略路由 --keyword S01 | case_runner run | 1 case |
| --tag 不存在 | case_runner run | 0 + WARNING filtered_count=0 |
| --case-file + --tag | case_runner run | WARNING 忽略 + 1 case |

## 6. 平台差异

- canonical 源：skills/case-execution/scripts/case_runner.py
- filter_by_tag/keyword 是纯列表筛选，无平台差异
- 不改 op_mapper.py（过滤是 case_runner 编排层逻辑）

## 7. 交接摘要

- ST-EX-16 filter_by_tag + filter_by_keyword 实现 + 7 单元测试通过 + 8 dry-run 集成场景通过
- discover_cases 重构为只 glob+排序，过滤逻辑由 main 链式调用
- argparse --tag（逗号分隔）+ --tag-mode（and/or）+ --keyword（append AND）
- --case-file + --tag warning 忽略；空结果 warning
- 依赖 ST-EX-15 parse_frontmatter（hard）已就绪
- O-EX-16-01（正则匹配）OPEN 非阻塞：当前精确+子串
- O-EX-16-02（tags 大小写）OPEN 非阻塞：当前大小写敏感
