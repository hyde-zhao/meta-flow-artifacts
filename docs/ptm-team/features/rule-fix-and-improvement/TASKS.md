---
cr_id: "CR-033"
artifact_type: "feature-tasks"
feature_id: "FE-EX-03"
feature_name: "规则固化与执行改进"
version: "1.0"
created_at: "2026-07-28T16:25:00+08:00"
author: "meta-se"
status: "draft"
source_design: "docs/features/rule-fix-and-improvement/DESIGN.md"
source_dev_plan: "process/DEVELOPMENT-PLAN.yaml"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md"
---

# FE-EX-03 规则固化与执行改进 Feature 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CP3 approved / CP4 PASS 后生成：Feature 级任务清单 TASK-EX-FE03-*；与 2 Story 关联；与 DEVELOPMENT-PLAN file_ownership 对齐 |

## 任务清单

> 任务 ID 格式 `TASK-EX-FE03-<seq>`。每项任务关联 Story 与 file_ownership（DEVELOPMENT-PLAN.yaml 真相源）。
> 实现顺序遵循 Story DAG：Wave1 ST-EX-02（规则块） -> Wave4 ST-EX-13（24 用例整改，依赖 ST-EX-08/15/17）。

### TASK-EX-FE03-01: install.py 规则块新增 ≥4 条规则（ST-EX-02，technical-note）

| 字段 | 值 |
|---|---|
| 任务 | render_ptm_te_rule_body 新增规则 9（TG 路由与 TREX_API_URL 注入）/ 规则 10（max_loss 参数化）/ 规则 11（ARP 预热双重保障）/ 规则 12（session 生命周期与 fw_logout 登出） |
| 关联 Story | ST-EX-02 |
| file_ownership | script/ptm_team/install.py#render_ptm_te_rule_body |
| 依赖 | 无（Wave 1，ST-EX-02 depends_on=[]） |
| 完成准则 | 规则 9/10/11/12 文本写入 render_ptm_te_rule_body（DESIGN §4.2）；f-string `${{ENV_VAR}}` 转义正确；managed block ID ptm-te-workflow 正则匹配；既有 8 条规则不破坏 |
| 验证 | static review（TEST-PLAN TP-FE03-01） |

### TASK-EX-FE03-02: 重装一致性验证（ST-EX-02，technical-note）

| 字段 | 值 |
|---|---|
| 任务 | install/uninstall/install 循环验证规则块 12 条不丢失 + op_mapper validate_mapping_consistency 通过 |
| 关联 Story | ST-EX-02 |
| file_ownership | script/ptm_team/install.py#render_ptm_te_rule_body（验证，不改代码） |
| 依赖 | TASK-EX-FE03-01；ST-EX-07（EXPECTED_OP_COUNT=22 协作） |
| 完成准则 | install -> uninstall -> install 循环后 CLAUDE.md managed block 含 12 条规则；case-execution skill 安装不丢失；op_mapper validate 通过（Gotcha FE03-1，HLD Gotcha #10） |
| 验证 | static（TEST-PLAN TP-FE03-02） |

### TASK-EX-FE03-03: 24 用例目录迁移（ST-EX-13，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 24 用例从 cases/upload/ 迁移到 cases/IPv4策略路由/配置管理/策略配置/ 三级结构；cases/upload/ 只保留 README.md 废弃说明 |
| 关联 Story | ST-EX-13 |
| file_ownership | ptm-te/cases/IPv4策略路由/**/*.md; ptm-te/cases/upload/README.md |
| 依赖 | ST-EX-08（ARP 预热引擎）/ ST-EX-15（用例结构化约定）/ ST-EX-17（环境解析层） |
| 完成准则 | 24 用例迁移到三级目录；每用例一文件；cases/upload/ 只保留 README.md 废弃说明，0 个 .md 用例文件（Gotcha FE03-2，HLD Gotcha #5） |
| 验证 | review + dry-run（TEST-PLAN TP-FE03-03/10） |

### TASK-EX-FE03-04: 24 用例重命名（ST-EX-13，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 24 用例文件重命名为 <编号>-<名称>.md（ADR-07 正则 ^PC-[A-Z0-9]+-\d+-\d+-\d+） |
| 关联 Story | ST-EX-13 |
| file_ownership | ptm-te/cases/IPv4策略路由/**/*.md |
| 依赖 | TASK-EX-FE03-03 |
| 完成准则 | 24 用例文件名编号正则匹配；frontmatter 用例编号列为唯一标识（Gotcha FE03-3，HLD Gotcha #6，ADR-07） |
| 验证 | dry-run + review（TEST-PLAN TP-FE03-04） |

### TASK-EX-FE03-05: 24 用例 frontmatter 16 列补全（ST-EX-13，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 24 用例 frontmatter 16 列补全（8 必填 + 8 可选，缺失可选列填 N/A）；tags/关键字列标注；known_issue 列标注 |
| 关联 Story | ST-EX-13 |
| file_ownership | ptm-te/cases/IPv4策略路由/**/*.md |
| 依赖 | TASK-EX-FE03-03 |
| 完成准则 | 16 列存在；8 必填非空；缺失可选列填 N/A（RA-012）；tags 列支持 --tag；关键字列支持 --keyword；known_issue 列按 DUT 行为差异标注（DQ-CP3-03）；忽略测试步骤/预期结果列（AGA-03=C，Gotcha FE03-4） |
| 验证 | review（TEST-PLAN TP-FE03-05） |

### TASK-EX-FE03-06: 24 用例 ARP 预热校验（ST-EX-13，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 24 用例含 TG 流量的 step 校验 warming_up:true 标注一致性 |
| 关联 Story | ST-EX-13 |
| file_ownership | ptm-te/cases/IPv4策略路由/**/*.md |
| 依赖 | TASK-EX-FE03-03; ST-EX-08（ARP 预热引擎，FE-EX-02） |
| 完成准则 | 24/24 用例 warming_up:true 合规（SM-EX-07）；缺失的补全；post_op 可由引擎自动补充（Gotcha FE03-8，ADR-06） |
| 验证 | review（TEST-PLAN TP-FE03-06） |

### TASK-EX-FE03-07: 24 用例 ${ENV.*} 引用改写（ST-EX-13，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 24 用例环境相关参数改写为 ${ENV.*} 占位符（9 类）；测试意图参数保持字面值 |
| 关联 Story | ST-EX-13 |
| file_ownership | ptm-te/cases/IPv4策略路由/**/*.md |
| 依赖 | TASK-EX-FE03-03; ST-EX-17（环境解析层，FE-EX-02） |
| 完成准则 | 环境相关参数（端口/IP/next_hop/URL）改写为 ${ENV.*}（DESIGN §5.5）；测试意图参数（src_ip/dst_ip/object_name/l4_protocol）保持字面值；dry-run 全部 ${ENV.*} 解析成功（Gotcha FE03-6，HLD Gotcha #12，ADR-09） |
| 验证 | dry-run（TEST-PLAN TP-FE03-07） |

### TASK-EX-FE03-08: 24 用例 dry-run 批量校验（ST-EX-13，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 24 用例迁移后 case_runner --dry-run 批量校验全部解析成功 |
| 关联 Story | ST-EX-13 |
| file_ownership | ptm-te/cases/IPv4策略路由/**/*.md（验证，不改代码） |
| 依赖 | TASK-EX-FE03-03/04/05/06/07; ST-EX-04（case_runner，FE-EX-02） |
| 完成准则 | 24/24 用例 dry-run 解析成功（DA-002）；frontmatter 16 列 + case_steps YAML + ${ENV.*} 全部正确 |
| 验证 | dry-run（TEST-PLAN TP-FE03-08） |

### TASK-EX-FE03-09: exec_v4.py 废弃标记（ST-EX-13，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | ptm-te/exec_v4.py 头部加 [DEPRECATED] 废弃标记 |
| 关联 Story | ST-EX-13 |
| file_ownership | ptm-te/exec_v4.py#废弃标记 |
| 依赖 | TASK-EX-FE03-03（迁移完成后） |
| 完成准则 | 头部 [DEPRECATED] 标记存在（DESIGN §5.6）；指向 case_runner 新入口；不删除文件（ADR-03，RA-009） |
| 验证 | review（TEST-PLAN TP-FE03-09） |

### TASK-EX-FE03-10: 规则 4 执行入口更新（ST-EX-13 协作 ST-EX-02）

| 字段 | 值 |
|---|---|
| 任务 | ST-EX-13 迁移完成后，更新 render_ptm_te_rule_body 规则 4 执行入口（cases/upload/ -> cases/三级结构） |
| 关联 Story | ST-EX-13（规则 4 更新）/ ST-EX-02（规则块函数） |
| file_ownership | script/ptm_team/install.py#render_ptm_te_rule_body |
| 依赖 | TASK-EX-FE03-01（规则块新增）; TASK-EX-FE03-03（目录迁移） |
| 完成准则 | 规则 4 文本更新为 cases/<特性>/<配置管理>/<策略配置>/*.md；旧 cases/upload/ 废弃说明（Gotcha FE03-7）；重装一致性重新验证 |
| 验证 | static（TEST-PLAN TP-FE03-01） |

## 任务依赖图

```
Wave 1:
TASK-EX-FE03-01 (ST-EX-02 规则块新增 ≥4 条)
   └─ TASK-EX-FE03-02 (ST-EX-02 重装一致性验证, 依赖 ST-EX-07 EXPECTED_OP_COUNT=22)

Wave 4 (依赖 Wave 2/3 完成):
TASK-EX-FE03-03 (ST-EX-13 目录迁移, 依赖 ST-EX-08/15/17)
   ├─ TASK-EX-FE03-04 (重命名)
   ├─ TASK-EX-FE03-05 (frontmatter 16 列)
   ├─ TASK-EX-FE03-06 (ARP 预热校验, 依赖 ST-EX-08)
   ├─ TASK-EX-FE03-07 (${ENV.*} 改写, 依赖 ST-EX-17)
   ├─ TASK-EX-FE03-09 (exec_v4.py 废弃标记)
   └─ TASK-EX-FE03-08 (dry-run 批量校验, 依赖 03/04/05/06/07 + ST-EX-04)

跨 Wave:
TASK-EX-FE03-10 (规则 4 更新, 依赖 01 + 03)
```

## 工作量汇总

| Story | Wave | 工作量 | 任务 |
|---|---|---|---|
| ST-EX-02 | 1 | S | TASK-EX-FE03-01/02 |
| ST-EX-13 | 4 | L | TASK-EX-FE03-03/04/05/06/07/08/09/10 |

**总计**：2 Story / 10 任务 / 2 Wave，工作量 1S + 1L（与 HLD §19 对齐：ST-EX-02=S, ST-EX-13=L）。

**跨 Feature 协作任务**：TASK-EX-FE03-02 依赖 ST-EX-07（FE-EX-02，EXPECTED_OP_COUNT=22）；TASK-EX-FE03-06 依赖 ST-EX-08（FE-EX-02，ARP 预热引擎）；TASK-EX-FE03-07 依赖 ST-EX-17（FE-EX-02，环境解析层）；TASK-EX-FE03-08 依赖 ST-EX-04（FE-EX-02，case_runner）。
