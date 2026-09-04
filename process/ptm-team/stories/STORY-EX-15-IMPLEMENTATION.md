---
cr_id: "CR-033"
story_id: "STORY-EX-15"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-15.md"
status: "implemented"
implemented_at: "2026-07-30T10:15:00+08:00"
author: "meta-dev"
---

# ST-EX-15 实现执行证据：用例结构化约定（parse_frontmatter + 16 列校验）

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行号 |
|---|---|---|---|---|
| _REQUIRED_FIELDS | skills/case-execution/scripts/case_runner.py | ST-EX-15 | 新增常量 | L273-L277 |
| _OPTIONAL_FIELDS | skills/case-execution/scripts/case_runner.py | ST-EX-15 | 新增常量 | L278-L281 |
| _ALL_16_FIELDS | skills/case-execution/scripts/case_runner.py | ST-EX-15 | 新增常量 | L282 |
| _FM_TAGS_KEYS | skills/case-execution/scripts/case_runner.py | ST-EX-15 | 新增常量 | L284 |
| _parse_case_id_from_filename | skills/case-execution/scripts/case_runner.py#_parse_case_id_from_filename | ST-EX-15 | 新增 | L287-L303 |
| parse_frontmatter | skills/case-execution/scripts/case_runner.py#parse_frontmatter | ST-EX-15 | 新增 | L305-L422 |
| SKILL.md 用例结构化约定 | skills/case-execution/SKILL.md | ST-EX-15 | 新增章节 | 相邻对象边界后、Gotchas 前 |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §复用既有 CASE_ID_RE（不重复定义 _CASE_ID_RE） | _parse_case_id_from_filename 用 CASE_ID_RE.match | ✓ 一致 |
| §16 列清单（8 必填 + 8 可选） | _REQUIRED_FIELDS(8) + _OPTIONAL_FIELDS(8) + _ALL_16_FIELDS | ✓ 一致 |
| DQ-05/LCQ-ST-EX-15-01 推荐方案 A（必填划分） | 名称/编号/级别/组网/预置/步骤/预期/类型 | ✓ 一致 |
| 必填缺失 -> error | errors.append(f"必填字段 '{fname}' 缺失") | ✓ 一致 |
| 可选缺失填 N/A（RA-012） | fields[fname] = "N/A" | ✓ 一致 |
| tags 第 17 字段可选，缺失填 [] | _FM_TAGS_KEYS 兼容 tags/标签；缺失 -> [] | ✓ 一致 |
| 关键词列逗号 split | str(raw_kw).split(",") + strip | ✓ 一致 |
| 冗余列忽略（AGA-03=C） | 测试步骤/预期结果只校验存在，不解析内容 | ✓ 一致 |
| frontmatter 缺失 -> error | errors 含 "frontmatter 缺失" | ✓ 一致 |
| case_id 从用例编号列 | fields.get("用例编号") | ✓ 一致 |
| 文件名编号正则匹配（Gotcha #6） | _parse_case_id_from_filename 复用 CASE_ID_RE | ✓ 一致 |
| parse_frontmatter 独立供过滤调用 | 不被 parse_case_file 调用，供 main 过滤路径调用 | ✓ 一致（设计允许选项） |

### 偏离设计证据的决策

1. **parse_frontmatter 独立、不被 parse_case_file 内部调用**：设计"建议 parse_case_file 内部调 parse_frontmatter 复用解析"，但选择独立方案以避免破坏既有 4 列 fixture（Wave 1/2 测试用例仅含 4 列，强制 16 列校验会标记 INVALID）。parse_frontmatter 在 main 过滤路径（--tag/--keyword）调用；parse_case_file 保持 4 列消费不变。frontmatter 解析开销极小，双重解析可接受。ST-EX-13 整改 24 用例对齐 16 列后可后续收紧。

2. **tags 键名兼容 tags + 标签**：设计 R-F-025 规定键名为 `tags`，但既有 fixture 用 `标签`（FM_TAGS）。_FM_TAGS_KEYS = ("tags", FM_TAGS) 兼容两者，ST-EX-13 整改后统一为 `tags`。

3. **CASE_ID_RE 格式**：既有 CASE_ID_RE = `^(PC-[A-Z0-9]+-\d+-\d+-\d+)`（1 字母数字段 + 3 数字段，如 PC-M4-01-01-01）。设计示例 `PC-COMB-M4-01-01` 不匹配此正则（COMB 与 M4 为独立段）。按任务要求复用既有 CASE_ID_RE，实际用例编号格式以 CASE_ID_RE 为准（ST-EX-13 整改时对齐）。

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| 单元测试 | /tmp/w3b-unit-tests.py | UT-15-01..06 + 4 extra |
| 16 列 dry-run 用例 | /tmp/w3b-test/cases/PC-M4-01-01-01-*.md | 全 16 列 + tags |

## 4. 最小实现切片

- _REQUIRED_FIELDS/_OPTIONAL_FIELDS/_ALL_16_FIELDS：16 列清单（DQ-05 推荐方案 A）
- _parse_case_id_from_filename：复用 CASE_ID_RE，返回 (case_id, case_name)
- parse_frontmatter：提取 --- 块 -> yaml.safe_load -> 校验 16 列 -> 解析 tags/keywords -> 返回结构化 dict
- SKILL.md 新增"用例结构化约定"章节：目录结构 + 命名 + 16 列 + tags + 冗余列忽略 + case_steps 保留

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| 语法检查 | python3 -c "import ast; ..." | SYNTAX OK |
| UT-15-01 16列完整 | unit_tests.py | PASS（16 fields, 0 errors） |
| UT-15-02 必填缺失 | unit_tests.py | PASS（errors 含"用例名称"） |
| UT-15-03 可选缺失填 N/A | unit_tests.py | PASS（首次创建版本=N/A） |
| UT-15-04 tags 列表 | unit_tests.py | PASS（5 tags） |
| UT-15-05 关键词 split | unit_tests.py | PASS（3 keywords） |
| UT-15-06 编号正则 | unit_tests.py | PASS（PC-M4-01-01-01 匹配） |
| tags 缺失 -> [] | unit_tests.py | PASS |
| 兼容 标签 键 | unit_tests.py | PASS（legacy-tag） |
| frontmatter 缺失 | unit_tests.py | PASS（errors 含 frontmatter） |
| case_id 提取 | unit_tests.py | PASS |
| dry-run 16列用例 | case_runner run --cases-dir | PASS（parse_frontmatter 过滤路径工作） |

## 6. 平台差异

- canonical 源：skills/case-execution/scripts/case_runner.py + SKILL.md
- parse_frontmatter 是纯解析（YAML safe_load），无平台差异
- 不改 24 用例 md（ST-EX-13 拥有），不改 op_mapper.py

## 7. 交接摘要

- ST-EX-15 parse_frontmatter + _parse_case_id_from_filename + 16 列常量实现 + 11 单元测试通过
- SKILL.md "用例结构化约定"章节新增（目录/命名/16列/tags/冗余忽略/case_steps 保留）
- LCQ-ST-EX-15-01 按推荐方案 A（8 必填 + 8 可选 + tags 第 17 字段）
- 复用既有 CASE_ID_RE，不重复定义 _CASE_ID_RE
- parse_frontmatter 独立供过滤路径调用（不入 parse_case_file，避免破坏既有 fixture）
- O-EX-15-01（英文字段别名）OPEN 非阻塞：当前中文字段名
