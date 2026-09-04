---
cr_id: "CR-033"
artifact_type: "feature-test-plan"
feature_id: "FE-EX-03"
feature_name: "规则固化与执行改进"
version: "1.0"
created_at: "2026-07-28T16:20:00+08:00"
author: "meta-se"
status: "draft"
source_design: "docs/features/rule-fix-and-improvement/DESIGN.md"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_scenarios: "docs/product/SCENARIOS-PTM-TE-EXEC.yaml"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md"
validation_mode: "static-only + dry-run-only + review-only"
covered_stories:
  - "ST-EX-02"
  - "ST-EX-13"
---

# FE-EX-03 规则固化与执行改进 Feature 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CP3 approved / CP4 PASS 后生成：Feature 级测试策略（static-only + dry-run-only + review-only）；2 Story × 验证点覆盖矩阵；重装一致性验证；24 用例整改校验；CP7 不真实设备写 |

## 1. 测试策略

### 1.1 validation_mode 声明

`validation_mode = static-only + dry-run-only + review-only`（HLD §13.1/§13.3，ADR-02）。

| 模式 | 适用 | CP7 可验证 | 说明 |
|---|---|---|---|
| static-only | install.py 规则块文本 / render_ptm_te_rule_body 函数 / managed block ID | 是 | code review + grep |
| dry-run-only | 24 用例迁移后 dry-run 批量解析 / ${ENV.*} 解析 | 是 | case_runner --dry-run |
| review-only | frontmatter 16 列 / 目录结构 / 重命名正则 / ARP 预热校验 / exec_v4.py 废弃标记 | 是 | 人工审查 |
| runtime（--execute） | 真实设备写操作 | **否**（CP7 N/A） | 需 runtime_authorization，超出 CR-033 |

**CP7 N/A 理由**：CR-033 不真实设备写（NO_PRODUCTION_WRITE）。规则块在运行时由 ptm-te agent 消费，CP7 只验证规则文本正确渲染与重装一致性，不验证 agent 运行时行为。

### 1.2 测试目标（量化）

| 指标 ID | 指标 | 目标值 | 验证方式 |
|---|---|---|---|
| TP-FE03-01 | 规则固化 | ≥4 条新增（共 12 条） | static review render_ptm_te_rule_body |
| TP-FE03-02 | 重装一致性 | 0 丢失（12 条规则 + case-execution skill） | install/uninstall/install 循环 |
| TP-FE03-03 | 24 用例目录迁移 | 24/24 迁移到三级结构 | review 目录 |
| TP-FE03-04 | 24 用例重命名 | 24/24 编号正则匹配 | dry-run + review |
| TP-FE03-05 | 24 用例 frontmatter 16 列 | 24/24（8 必填 + 8 可选） | review |
| TP-FE03-06 | 24 用例 ARP 预热校验 | 24/24 warming_up 合规 | review |
| TP-FE03-07 | 24 用例 ${ENV.*} 改写 | 24/24 解析成功 | dry-run resolve_env_refs |
| TP-FE03-08 | 24 用例 dry-run | 24/24 全部解析成功 | dry-run 批量 |
| TP-FE03-09 | exec_v4.py 废弃标记 | 1 个废弃标记头部 | review |
| TP-FE03-10 | cases/upload 废弃 | 只保留 README.md，0 个 .md 用例 | review |

### 1.3 测试依据来源（理论依据）

- FMEA 故障模式分析（重装回退是已知故障模式，规则+引擎双重保障，HLD §21.7）
- JTBD（测试执行工程师"工作"是用例执行，零代码新增用例，HLD §21.7）
- ISTQB 测试结果分级（known_issue 四态，HLD §21.7）

## 2. 测试覆盖矩阵（Story × 验证点）

| Story | 验证点 | 验证模式 | 预期 | 关联场景 |
|---|---|---|---|---|
| ST-EX-02 | render_ptm_te_rule_body 新增 ≥4 条规则 | static | 规则 9/10/11/12 文本存在 | SCN-EX-18 |
| ST-EX-02 | 规则 9 TG 路由与 TREX_API_URL 注入 | static | 描述 _build_exec_env + ptm-atomic 约束 | SCN-EX-18 |
| ST-EX-02 | 规则 10 max_loss 参数化 | static | 禁止硬编码 --max-loss | SCN-EX-18 |
| ST-EX-02 | 规则 11 ARP 预热双重保障 | static | warming_up 强制 post_op + 引擎兜底 | SCN-EX-05,18 |
| ST-EX-02 | 规则 12 session 生命周期与 fw_logout | static | fw_logout 登出 + 降级 session 清理 | SCN-EX-19 |
| ST-EX-02 | managed block ID ptm-te-workflow | static | 正则匹配成功 | SCN-EX-18 |
| ST-EX-02 | 重装一致性（install/uninstall/install 循环） | static | 12 条规则不丢失 | SCN-EX-18 |
| ST-EX-02 | op_mapper validate_mapping_consistency | static | EXPECTED_OP_COUNT=22 通过（ST-EX-07 协作） | SCN-EX-19 |
| ST-EX-13 | 24 用例目录迁移到三级结构 | review | cases/IPv4策略路由/配置管理/策略配置/*.md | SCN-EX-20,21 |
| ST-EX-13 | 24 用例重命名（编号正则） | dry-run | ^PC-[A-Z0-9]+-\d+-\d+-\d+ 匹配 | SCN-EX-21 |
| ST-EX-13 | 24 用例 frontmatter 16 列（8 必填+8 可选） | review | 缺失可选列填 N/A | SCN-EX-21 |
| ST-EX-13 | 24 用例 tags/关键字列标注 | review | 支持 --tag/--keyword | SCN-EX-22 |
| ST-EX-13 | 24 用例 known_issue 标注（DUT 行为差异） | review | known_issue 列正确标注 | SCN-EX-09 |
| ST-EX-13 | 24 用例 ARP 预热校验（warming_up:true） | review | 24/24 合规 | SCN-EX-05 |
| ST-EX-13 | 24 用例 ${ENV.*} 引用改写 | dry-run | 9 类占位符解析成功 | SCN-EX-27 |
| ST-EX-13 | 24 用例 dry-run 批量解析 | dry-run | 24/24 解析成功 | SCN-EX-20 |
| ST-EX-13 | 测试意图参数保持字面值 | review | src_ip/dst_ip/object_name 不改写 | SCN-EX-27 |
| ST-EX-13 | exec_v4.py 废弃标记 | review | 头部 [DEPRECATED] 标记存在 | SCN-EX-20 |
| ST-EX-13 | cases/upload 废弃（只保留 README.md） | review | 0 个 .md 用例文件 | SCN-EX-20 |
| ST-EX-13 | 规则 4 执行入口更新（cases/三级） | static | 规则 4 文本更新 | SCN-EX-18 |

## 3. 验证场景映射（SCN-EX -> Story）

| 场景 ID | 场景 | 关联 Story | 验证模式 |
|---|---|---|---|
| SCN-EX-05 | ARP 预热校验 | ST-EX-02（规则）/ ST-EX-13（用例） | review |
| SCN-EX-09 | known_issue 标注 | ST-EX-13 | review |
| SCN-EX-18 | 重装一致性 + 规则块 | ST-EX-02 | static |
| SCN-EX-19 | fw_logout 规则（规则 12） | ST-EX-02 | static |
| SCN-EX-20 | 24 用例迁移 + exec_v4.py 废弃 | ST-EX-13 | dry-run + review |
| SCN-EX-21 | 用例结构化（目录/命名/frontmatter） | ST-EX-13 | review |
| SCN-EX-22 | tags/关键字标注 | ST-EX-13 | review |
| SCN-EX-27 | ${ENV.*} 改写 | ST-EX-13 | dry-run |

## 4. 测试分层计划

| 层级 | 范围 | 方式 | 责任 |
|---|---|---|---|
| L1 静态检查 | render_ptm_te_rule_body 规则文本 / managed block ID / validate_mapping_consistency | code review + grep | meta-qa |
| L2 dry-run | 24 用例迁移后 dry-run 批量解析 / ${ENV.*} 解析 / 编号正则匹配 | case_runner --dry-run | meta-qa |
| L3 产物审查 | frontmatter 16 列 / 目录三级结构 / 重命名 / ARP 预热校验 / exec_v4.py 废弃 / cases/upload 废弃 | 人工审查 | meta-qa |
| L4 重装一致性 | install/uninstall/install 循环 / 12 条规则不丢失 / op_mapper validate | install.py 循环 | meta-qa |
| L5 runtime（N/A） | 真实设备 --execute / agent 运行时规则消费 | 需 runtime_authorization | 后续 CR |

## 5. fixture / dry-run 计划

| fixture | 用途 | 来源 |
|---|---|---|
| fixtures/env-link3.yaml | link3 环境 env-file（24 用例 ${ENV.*} 解析校验） | meta-qa 构造（与 FE-EX-02 共享） |
| fixtures/env-link4.yaml | link4 环境 env-file（多环境执行校验） | meta-qa 构造（与 FE-EX-02 共享） |
| fixtures/expected-rule-block.txt | 12 条规则预期文本（重装一致性对比） | meta-qa 构造 |

dry-run 命令模板（24 用例迁移后）：
```bash
python skills/case-execution/scripts/case_runner.py run \
  --cases-dir ptm-te/cases/IPv4策略路由/ \
  --devices-yaml fixtures/devices.test.yaml \
  --env-file fixtures/env-link3.yaml \
  --dry-run --output-dir runs/test-run-id/
```

重装一致性命令：
```bash
python script/ptm_team/install.py install ptm-te --component full --workspace <ws>
python script/ptm_team/install.py uninstall ptm-te --workspace <ws>
python script/ptm_team/install.py install ptm-te --component full --workspace <ws>
# 校验 CLAUDE.md managed block 含 12 条规则
# 校验 .claude/skills/case-execution/ 存在
python skills/policy-route-execution/scripts/op_mapper.py validate
```

## 6. 风险与未覆盖项

| 风险 ID | 风险 | CP7 处理 |
|---|---|---|
| R-FE03-01 | 规则块运行时由 agent 消费，CP7 不验证 agent 行为 | CP7 N/A，只验证规则文本渲染正确 |
| R-FE03-02 | 24 用例 known_issue 标注完整性 | 整改时同步检查（DQ-CP3-03，O-FE03-01） |
| R-FE03-03 | fw_logout 未暴露时规则 12 降级触发条件 | 安装前 `ptm-atomic show fw_logout` 验证（O-FE03-02） |
| R-FE03-04 | 规则 4 执行入口更新依赖 ST-EX-13 迁移完成 | ST-EX-02 先保留旧文本，ST-EX-13 迁移后更新（Gotcha FE03-7） |
| R-FE03-05 | 24 用例 ${ENV.*} 改写与 ST-EX-17 解析规则不一致 | dry-run 校验全部 ${ENV.*} 解析成功（Gotcha FE03-6） |

## 7. 阶段决策

| 决策 | 内容 |
|---|---|
| CP7 结论预期 | PASS_WITH_RISK（agent 运行时未覆盖为已知风险，static+dry-run+review 全过） |
| 风险接受项 | R-FE03-01/02/03 进 CP8 risk acceptance |
| 后续 CR 候选 | agent 运行时规则消费验证 CR（含 runtime_authorization） |
