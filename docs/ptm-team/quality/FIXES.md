---
story_range: "CR-030 (14 Stories)"
fixes_scope: "CP7 static review findings"
generated_by: "meta-qa"
generated_at: "2026-07-16T15:00:00+00:00"
source_cr: "CR-030"
---

# ptm-tse CR-030 回修输入

> 本文件列出 CP7 静态审查中发现的问题及建议修复方案。所有问题均不阻断 CP8 推进，但建议在后续迭代中处理。

---

## 回修清单

### FIX-01: 补写 4 份缺失的 CP6 检查点

- **优先级**：HIGH
- **影响 Story**：ST-RA-05.2-CLEAN, ST-RA-05.3-ANALYZE, ST-RA-06.1-DETECT, ST-RA-06.2-REFRESH
- **问题**：实现产物存在但无 CP6 编码完成检查点
- **回修方案**：
  1. 参考已有的 10 份 CP6 检查点格式（如 `CP6-ST-RA-05.1-INGEST-CODING-DONE.md`）
  2. 每份 CP6 需包含：Entry Criteria、实现对象清单、设计契约映射、测试/Fixture 计划、最小实现切片、本地验证、平台差异检查、QA/Review/Doc 交接摘要
  3. 文件命名格式：`CP6-<story_id>-CODING-DONE.md`
  4. 存放路径：`process/checks/`
- **替代方案**：host-orchestrator 在 CP8 Decision Brief 中显式豁免，并记录豁免原因
- **责任人**：meta-dev（或 host-orchestrator 豁免）

### FIX-02: itr-ticket-ingestion 追加 Gotchas 章节

- **优先级**：MEDIUM
- **影响文件**：`skills/itr-ticket-ingestion/SKILL.md`
- **问题**：缺少 Gotchas 章节（违反 CLAUDE.md 规则 §10）
- **回修方案**：在文件末尾（修订记录之前）追加 `## Gotchas` 章节，至少包含：
  1. 不要在 allowlist 中配置通配符 URL
  2. 不要手动修改 `data/snapshots/` 下的快照文件
  3. 不要在分页失败时覆盖已成功的页
  4. 不要跳过 allowlist 校验直接发起 HTTP 请求
  5. 不要在事务外写入数据
  6. 不要忽略 conflict_queue
  7. 不要假设 source_ticket_id 永远存在
- **责任人**：meta-dev
- **参考**：`skills/improvement-tracker/SKILL.md` 的 Gotchas 格式（22 条，编号列表 + 详细说明）

### FIX-03: reverse-analysis 追加 Gotchas 章节

- **优先级**：MEDIUM
- **影响文件**：`skills/reverse-analysis/SKILL.md`
- **问题**：缺少 Gotchas 章节（违反 CLAUDE.md 规则 §10）
- **回修方案**：在文件末尾（修订记录之前）追加 `## Gotchas` 章节，至少包含：
  1. 不要在 valid_count < 3 时输出 confirmed 根因
  2. 不要将无可信分母的数据标注为密度指标
  3. 不要跨 ticket 借用证据
  4. 不要从 raw_json 提取未映射数据
  5. 不要自动调用 reviewer_publish_analysis_run
  6. 不要混淆 S1 和 S2 管线路由
  7. 不要从 failed 状态恢复 analysis_run
  8. 不要将无控制证据的 escape layer 标记为 confirmed
  9. 不要将 P2 问题单自动推进分析
  10. 不要在规则版本变更时使用增量重算
- **责任人**：meta-dev
- **参考**：`skills/improvement-tracker/SKILL.md` 的 Gotchas 格式

### FIX-04: 对齐 STATE 与实际工作流状态

- **优先级**：LOW
- **影响文件**：`process/state/STATE.current.json`
- **问题**：STATE 显示 CP5 pending，实际已推进到 CP7
- **回修方案**：由 host-orchestrator 在 CP8 终验前更新 STATE，或在 CP8 Decision Brief 中标注 bypass 原因
- **责任人**：host-orchestrator

### FIX-05: 统一禁止规则统计描述

- **优先级**：LOW
- **影响文件**：`skills/reverse-analysis/SKILL.md` 修订记录
- **问题**：v1.4 修订记录暗示 19 项，实际禁止事项表 23 项
- **回修方案**：在修订记录 v1.4 条目中将 "P-12 至 P-19" 改为 "P-12 至 P-15（含越权检测 + 审计日志 + 拒绝矩阵），禁止事项表共 23 项"，或统一计数方式
- **责任人**：meta-dev

---

## 修复优先级建议

| 优先级 | 修复项 | 建议窗口 |
|--------|--------|---------|
| 发布前 | FIX-01（或豁免） | CP8 终验前 |
| 发布后首个迭代 | FIX-02, FIX-03 | 后续 CR 台账 |
| 下次状态更新时 | FIX-04, FIX-05 | 便利性修复 |

## 不接受自动修复的约束

以下操作**不得**由 meta-qa 自动执行：
- 不修改 Skill 文件正文
- 不创建 CP6 检查点文件
- 不修改 STATE.current.json
- 不修改修订记录
- 所有修复必须由对应责任方（meta-dev 或 host-orchestrator）审批后执行
